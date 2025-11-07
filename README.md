
High-Performance Next.js 14 Dashboard
        |
| ------------------------ | :--------------------------------------------------------------------------- |
| **8** Pages              | **80+** Pages                                                                |
| ✔ Custom Authentication  | ✔ Authentication with **Amplify**, **Auth0**, **Firebase** and **Supabase**  |
| -                        | ✔ Vite Version                                                               |
| -                        | ✔ Dark Mode Support                                                          |
| -                        | ✔ Complete Users Flows                                                       |
| -                        | ✔ Premium Technical Support                                                  |


## File Structure

Within the download you'll find the following directories and files:

```
┌── .editorconfig
├── .eslintrc.js
├── .gitignore
├── CHANGELOG.md
├── LICENSE.md
├── next-env.d.ts
├── next.config.js
├── package.json
├── README.md
├── tsconfig.json
├── public
└── src
	├── components
	├── contexts
	├── hooks
	├── lib
	├── styles
	├── types
	└── app
		├── layout.tsx
		├── page.tsx
		├── auth
		└── dashboard
```

This project is an implementation of a real-time, high-performance dashboard capable of rendering 10,000+ data points at 60fps. It is built from scratch using Next.js 14 (App Router), TypeScript, and the raw Canvas API for charting, as per the assignment requirements.

🚀 Getting Started

Install Dependencies:

npm install


Run the Development Server:

npm run dev


Open http://localhost:3000/dashboard to view the dashboard.

✨ Features

Real-Time Data Stream: A Server-Sent Events (SSE) API route (app/api/data/route.ts) streams 10,000 data points on load and pushes a new update every 100ms.

High-Performance Line Chart: The line chart (components/charts/LineChart.tsx) is rendered using the <canvas> element.

requestAnimationFrame Loop: Rendering is decoupled from React's render cycle and runs in a rAF loop for maximum smoothness.

High-DPI Scaling: Canvas is properly scaled for high-DPI (Retina) displays.

Efficient State Management: Data is provided via a Context (DataProvider.tsx) that uses useTransition to apply state updates without blocking the UI.

Live Performance Monitor: A component (components/ui/PerformanceMonitor.tsx) tracks and displays real-time FPS and JS Heap Memory usage.

Next.js 14 App Router:

Server Components: The main layout (app/dashboard/layout.tsx) is a Server Component.

Client Components: The interactive dashboard (app/dashboard/page.tsx) and all charts/controls are Client Components ('use client').

Route Handlers: The data API is built as a Route Handler.

(Stubbed) Features:

Bar Chart, Scatter Plot, Heatmap

Data Filtering & Aggregation

Time Range Selection

Virtualized Data Table

🖥️ Performance Testing

Run the application in production mode for accurate measurements:

npm run build
npm run start


Open http://localhost:3000/dashboard in your browser.

Use the Live Performance Monitor in the UI to observe FPS.

Open your browser's Developer Tools:

Performance Monitor (Chrome): Open DevTools > (More Tools) > Performance monitor. Watch the "JS heap size" and "CPU usage".

Lighthouse: Run a Lighthouse audit to check Core Web Vitals.

Performance Profiler: Record a performance profile while interacting with the (future) zoom/pan controls to check for long tasks or dropped frames.

Compatible Browsers

Chrome (latest)

Firefox (latest)

Safari (latest)

Edge (latest)
