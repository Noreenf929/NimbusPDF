FRONTEND bugs
1. vite.config.js renamed to vite.config.mjs as its a module. npm ci was giving errors. Status: Fixed.
2. Inside AISidebar.svelte, in function quickAction() the api.post() response was expecting reply like in the send() but backend return a result key instead of reply. Status: Fixed.
3. In the AISidebar.svelte, clear functionaility is not working as expected. Status: Fixed. Added DELETE /ai/history/:doc_id backend endpoint; clearChat() now calls it before wiping local state so history doesn't reappear on reload.
4. When a new PDF is uploaded, page shows 401 error first but when reloaded, everything looks good. Status: Fixed. Root cause same as bug 5 — pdfStore.load() ran during SSR before the session cookie was available. Moving to onMount resolved it.
5. After opening a PDF, if the viewer is refresh, page crashes with following error : (Error: Cannot call `fetch` eagerly during server-side rendering with relative URL (/api/pdfs/59110104-5e92-4b77-825f-62d8ae725fcd) — put your `fetch` calls inside `onMount` or a `load` function instead). Status: Fixed. Moved `pdfStore.load(docId)` from a reactive statement (`$:`) to `onMount` in viewer/[docId]/+page.svelte so it only runs client-side.
6. Search functionality when the PDF is opened is not working. Status: Fixed. PDFViewer now reacts to $pdfStore.searchQuery; uses PDF.js getTextContent() to fetch text items per page, converts their PDF-space coordinates to viewport CSS pixels via convertToViewportRectangle(), and renders a yellow search overlay. First matching page is scrolled into view. rawTextCache is zoom-invariant (PDF space), and search rects are recomputed after every zoom re-render.
7. PDF rendering on Hi res displays like mac is blurry. Status: Fixed. renderPageNum() now reads window.devicePixelRatio and sets canvas.width/height to floor(viewport * dpr) physical pixels, then scales the 2D context by dpr before rendering. Canvas CSS size stays at logical viewport dimensions so layout is unchanged.
8. PDF doesn't support selected of text of highlighting sections of text. Status: Pending. 
9. Unable to upload pdf after bug fixes. Status: Fixed. Max upload limit was set to 2MB (default for axum).
10. Proper messages weren't flowing to front end in case of upload failure. Status: Fixed.
11. When user selects to add the document to long term memory, the 401 error comes. But when page is refreshed the pdf appear on the main screen and then open successfully upon clicking. If the ue declines to add document to long term memory, it successfully open on the first go.  Status: Fixed. auth.init() IS called in the layout. The problem is clear now: /auth/me always returns a JSON object (never null), so $auth.user is always truthy — even for anonymous users. Every anonymous user sees the memory
  confirm dialog, then gets a 401.
12. Delete PDF feature wasn't present. Status: Implemented.
13. Unable to select words/sentences inside PDF viewer for searching or asking AI for assistance. Status: Fixed. Added PDF.js TextLayer (pdfjs-dist 4.x API) rendered as a transparent positioned-span overlay on each page canvas. TextLayer instances are cancelled and recreated on zoom re-render and cleaned up on component destroy. Minimal text layer CSS added to app.css. Highlight overlay given explicit z-index:3 to remain above the text layer (z-index:2) while search overlay stays at z-index:5.
14. The highlight option opens up but fill box color of the underlying text doesn't change. Status: Fixed. Two root causes resolved: (1) Backend Highlight struct had range_start/range_end fields that the frontend never sent — replaced with rects: Vec<HighlightRect> and created_at: Option<String> to match the frontend payload. (2) svelte:window on:mouseup fired before the color-button click handler, calling dismissPicker() and clearing pendingSelection before applyHighlight could run — fixed by returning early in onMouseUp when the picker is already visible, relying on the backdrop div for outside-click dismissal.
15. During an anonymous session, 
  a. Save the model output history and chat history in respective markdown file just next to pdf files.
  b. The clear chat option to clear the content inside those file. 
  c. There should also be a clear cache button on the home to clear all web and backend session data and start a fresh session.
  Status: Implemented. append_chat_history() now also writes chat_history.md (timestamped Q&A entries) next to the PDF. run_quick_ai_task() calls new append_model_output_md() to append summary/keypoints results to model_output.md. clear_history endpoint zeroes both markdown files alongside chat_history.json. Added POST /api/auth/clear-cache backend endpoint (deletes session file + anonymous data dir for anonymous users, expires cookie). Home page gained a "Clear Cache" button that confirms, calls the endpoint, and reloads to a fresh session.
16. Inside the AISidebar, there should be option to override the default model. This list of models should appear as dropdown list in the component and should be fetched from backend ollama/openAI compatible server. Status: Implemented. Added GET /api/ai/models backend endpoint: loads user AI config, derives the models URL by replacing the endpoint path with /v1/models (works for Ollama ≥0.3 and OpenAI), returns { models: [...] }. ChatRequest and DocRequest now accept an optional model field. When set, the per-request model overrides the configured default for that call. AISidebar fetches models on mount and renders a dropdown above the action buttons; selecting a model passes it with every chat / summary / keypoints request.
17. The default model settings (endpoint, modelname, token) should be copied for every new session from backend configuration file "default.toml". If the user want, they can overwrite that on-demand. Even after clear cache, the new session should get default model settings. Status: Pending.
18. The pinch zoom-in/zoom-out (from trackpad), scales the entire page instead of just the pdf. It should also work with mouse (Ctrl/Cmd + wheel up/down). Status: Pending.
19. The settings page has no dedicated back button. Status: Pending.
20. The text selection on the PDF has following bugs:
  a. When selecting highligh option, its doesn't do anything right way. when the pdf is viewed again, those highlights do appear. 
  b. Highlighting is currently zoom agnostic, messeses up the highlight placement when zoom is changed. 
  c. Text selection is buggy. Doesn't cover the full text properly. For examples if the word is "language", the "g" letter is only half covered. 
  Status: Pending.
21. Still in the anonymous session, the previous chat history of the pdf is not shown. Status: Pending.
22. Implement a page index (editable) and page change button. When user enters a specific page number in the page index, the pdf should go to that page. Status: Pending.
22. The notes section has following bugs:
  a. Shows notes for the last page number only. Notes should be for every page. Status: Pending.


BACKEND bugs
1. In model configuration, the frontend expected json output but inside backend/src/routes/ai.rs:save_ai_config returned only status code. Status: Fixed.