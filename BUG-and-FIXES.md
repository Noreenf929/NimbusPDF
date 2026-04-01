FRONTEND bugs
1. vite.config.js renamed to vite.config.mjs as its a module. npm ci was giving errors. Status: Fixed.
2. Inside AISidebar.svelte, in function quickAction() the api.post() response was expecting reply like in the send() but backend return a result key instead of reply. Status: Fixed.
3. In the AISidebar.svelte, clear functionaility is not working as expected. Status: Pending.
4. When a new PDF is uploaded, page shows 401 error first but when reloaded, everything looks good. Status: Pending.
5. After opening a PDF, if the viewer is refresh, page crashes with following error : (Error: Cannot call `fetch` eagerly during server-side rendering with relative URL (/api/pdfs/59110104-5e92-4b77-825f-62d8ae725fcd) — put your `fetch` calls inside `onMount` or a `load` function instead). Status: Pending.

BACKEND bugs
1. In model configuration, the frontend expected json output but inside backend/src/routes/ai.rs:save_ai_config returned only status code. Status: Fixed.