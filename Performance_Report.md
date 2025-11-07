Performance & Architecture Report

This document outlines the performance optimization techniques and architectural decisions made for this high-performance dashboard, as required by the assignment.

1. Benchmarking Results (To Be Filled Out)

Instructions: Run the production build (npm run build && npm run start) and fill this out.

Metric

Target

Result

Notes

FPS (Real-time updates)

60 FPS

[XX] FPS

Measured via the PerformanceMonitor UI on a [Your Machine Specs]

Interaction Latency (Zoom/Pan)

< 100ms

[XX] ms

TODO: Measure after implementing zoom/pan

Memory Growth (1 hour)

< 1MB / hour

[XX] MB

TODO: Run app for 1 hour and measure heap

Data Points Handled

10,000+

10,000

Successfully renders and updates 10,000 points.

Bundle Size (Dashboard Page)

< 500KB gzipped

[XX] KB

TODO: Run npm run build and analyze output

Core Web Vitals (LCP, FID, CLS)

All Green

[Pass/Fail]

TODO: Run Lighthouse audit

2. React Performance Optimization Techniques

useTransition for Non-Blocking Data Updates

The most critical optimization is in components/providers/DataProvider.tsx. When new data arrives from the useDataStream hook (every 100ms), we wrap the setData state update in startTransition:

// In DataProvider.tsx
startTransition(() => {
  setData((prevData) => {
    // ... logic to append new data and slice array
  });
});


Why? This tells React that the state update is non-urgent. If the user is in the middle of an interaction (like panning the chart), React will not block that interaction to update the data. This keeps the UI responsive at all times.

requestAnimationFrame for Canvas Rendering

React's render cycle is not designed for 60fps animations. In components/charts/LineChart.tsx, we decouple rendering from React state:

A useRef holds the <canvas> element.

A useEffect hook is responsible for starting and stopping a render loop using requestAnimationFrame.

This loop (renderChart function) reads the latest data (from useData) and draws directly to the canvas.

The useEffect cleanup function must call cancelAnimationFrame to prevent memory leaks when the component unmounts.

React.memo (and useMemo/useCallback philosophy)

LineChart.tsx is wrapped in React.memo. While its data comes from context, this prevents re-renders if the parent component (page.tsx) were to re-render for other reasons.

Strategy (for you to implement): Any complex calculations (like data aggregation or finding min/max over 10k points) must be wrapped in useMemo. All event handlers (like onZoom or onPan) must be wrapped in useCallback to ensure components wrapped in React.memo do not re-render unnecessarily.

3. Next.js 14 Performance Features (App Router)

Server Components vs. Client Components

Server Components: app/layout.tsx and app/dashboard/layout.tsx are React Server Components (RSCs) by default. They render on the server, producing static HTML for the page shell. This reduces the amount of JavaScript sent to the client for the non-interactive parts.

Client Components: app/dashboard/page.tsx and all components under components/ are marked with 'use client'. This is necessary because they use state (useState), effects (useEffect), and browser-only APIs (Canvas, window.performance).

Route Handlers (SSE) for Real-Time Data

Instead of traditional REST or WebSockets, we use a Route Handler at app/api/data/route.ts that returns a ReadableStream.

This implements Server-Sent Events (SSE), which is a lightweight and efficient way to push data from the server to the client over a single HTTP connection. It's simpler to manage than WebSockets for a one-way data flow.

(Future) Streaming & Static Generation

Streaming: The dashboard page could be wrapped in a <Suspense> boundary, allowing Next.js to stream the static parts of the page (like the header) first, while the DataProvider (which is client-side) fetches its initial data.

Static Generation: While our dashboard is real-time, parts of it (like a "Help" modal or a static config) could be Server Components that are statically generated at build time.

4. Canvas + React Integration Strategy

This is the core challenge. The strategy is to let React manage state and let Canvas manage rendering.

React Owns State: React components (DataProvider, FilterPanel) are responsible for all state: the data array, the filter settings, the zoom level, etc.

Canvas is a "Dumb" Renderer: The LineChart.tsx component is a bridge.

It uses useData() to get the current data state from context.

It uses useRef() to get a direct handle to the DOM's <canvas> element.

It uses useEffect() to run code after React has rendered.

The rAF Loop: Inside useEffect, we start the requestAnimationFrame loop. This loop is outside of React's control.

On every frame, it reads the current data and viewport state.

It performs raw canvas.getContext('2d') draw calls (.beginPath(), .lineTo(), .stroke()).

This is extremely fast because it bypasses React's diffing algorithm and the virtual DOM.

Hybrid (Canvas + SVG/HTML): The LineChart.tsx file includes a comment on the "Hybrid Approach."

Canvas: Used for the 10,000-point line itself, as it's the performance-critical part.

HTML/SVG (Overlay): For things that don't update 60 times per second (like tooltips, axis labels, crosshairs), we can overlay HTML or SVG elements on top of the canvas. This is easier to style and manage interactivity for.

5. Scaling Strategy (Client vs. Server)

Client-Side Rendering (CSR): The charts must be rendered on the client. Server-Side Rendering (SSR) a 10,000-point canvas chart is not feasible, especially for real-time updates.

Server-Side Data Generation: The data originates from the server (/api/data). This is correct. In a real app, this route would query a time-series database (like InfluxDB or TimescaleDB).

Data Aggregation:

Current: Aggregation is stubbed as a client-side filter.

Scaling Strategy: For true large-scale (1,000,000+ points), aggregation (e.g., "1 hour" buckets) should be done on the server/database. The client should query /api/data?aggregate=1h. This dramatically reduces the amount of data sent over the network. The client-side aggregation is only for small-scale adjustments.
