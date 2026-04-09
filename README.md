<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Wysyłki — Panel Zarządzania</title>
  <link href="[fonts.googleapis.com](https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap)" rel="stylesheet">
  <link href="[cdnjs.cloudflare.com](https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css)" rel="stylesheet">
  <script src="[cdn.jsdelivr.net](https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2)"></script>
  <style>
    :root {
      --bg: #060a10; --bg-card: #0c1119; --bg-elevated: #111827; --bg-hover: #151d2e;
      --border: #1e293b; --border-light: #334155; --text: #f1f5f9; --text-secondary: #94a3b8;
      --text-muted: #64748b; --text-dim: #475569; --accent: #3b82f6; --accent-hover: #2563eb;
      --accent-light: rgba(59,130,246,0.12); --accent-glow: rgba(59,130,246,0.25);
      --success: #22c55e; --success-light: rgba(34,197,94,0.1); --warning: #f59e0b;
      --warning-light: rgba(245,158,11,0.1); --danger: #ef4444; --danger-light: rgba(239,68,68,0.1);
      --purple: #8b5cf6; --purple-light: rgba(139,92,246,0.1); --teal: #06b6d4;
      --teal-light: rgba(6,182,212,0.1); --radius: 8px; --radius-lg: 12px;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; }
    html, body { height: 100%; overflow: hidden; }
    body { background: var(--bg); color: var(--text); font-family: 'Inter', system-ui, -apple-system, sans-serif; font-size: 14px; line-height: 1.5; -webkit-font-smoothing: antialiased; }
    ::-webkit-scrollbar { width: 5px; } ::-webkit-scrollbar-track { background: transparent; }
    ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }
    ::-webkit-scrollbar-thumb:hover { background: var(--text-dim); }
    @keyframes fadeInUp { from { opacity:0; transform:translateY(16px); } to { opacity:1; transform:translateY(0); } }
    @keyframes fadeIn { from { opacity:0; } to { opacity:1; } }
    @keyframes slideRight { from { opacity:0; transform:translateX(40px); } to { opacity:1; transform:translateX(0); } }
    @keyframes scaleUp { from { opacity:0; transform:scale(0.95); } to { opacity:1; transform:scale(1); } }
    @keyframes sheetUp { from { transform:translateY(100%); } to { transform:translateY(0); } }
    @keyframes barGrow { from { width:0; } }
    .anim-up { animation: fadeInUp 0.45s cubic-bezier(0.16,1,0.3,1) both; }
    .anim-fade { animation: fadeIn 0.4s ease both; }
    .anim-right { animation: slideRight 0.4s cubic-bezier(0.16,1,0.3,1) both; }
    .anim-scale { animation: scaleUp 0.35s cubic-bezier(0.16,1,0.3,1) both; }
    .d1{animation-delay:.04s}.d2{animation-delay:.08s}.d3{animation-delay:.12s}
    .d4{animation-delay:.16s}.d5{animation-delay:.2s}.d6{animation-delay:.24s}
    .login-screen { position:fixed; inset:0; z-index:200; display:grid; place-items:center; background:var(--bg); transition:opacity 0.5s cubic-bezier(0.16,1,0.3,1),transform 0.5s cubic-bezier(0.16,1,0.3,1); }
    .login-screen.hidden { opacity:0; transform:scale(1.04); pointer-events:none; }
    .login-card { width:100%; max-width:400px; background:var(--bg-card); border:1px solid var(--border); border-radius:16px; padding:40px 32px; box-shadow:0 25px 60px rgba(0,0,0,0.5); animation:fadeInUp 0.7s cubic-bezier(0.16,1,0.3,1); }
    .login-header { text-align:center; margin-bottom:32px; }
    .login-icon { width:56px; height:56px; margin:0 auto 16px; background:linear-gradient(135deg,var(--accent),#6366f1); border-radius:14px; display:flex; align-items:center; justify-content:center; font-size:24px; color:white; box-shadow:0 8px 24px rgba(59,130,246,0.3); }
    .login-header h1 { font-size:22px; font-weight:700; margin-bottom:4px; }
    .login-header p { color:var(--text-muted); font-size:13px; }
    .field { margin-bottom:16px; }
    .field label { display:block; font-size:12px; font-weight:600; color:var(--text-muted); margin-bottom:6px; text-transform:uppercase; letter-spacing:0.5px; }
    .input { width:100%; padding:12px 14px; background:var(--bg); border:1px solid var(--border); border-radius:var(--radius); color:var(--text); font-size:14px; font-family:inherit; outline:none; transition:border-color 0.2s,box-shadow 0.2s; }
    .input:focus { border-color:var(--accent); box-shadow:0 0 0 3px var(--accent-light); }
    .input::placeholder { color:var(--text-dim); }
    select.input { cursor:pointer; } select.input option { background:var(--bg-card); }
    textarea.input { resize:vertical; min-height:70px; }
    .remember-row { display:flex; align-items:center; gap:8px; margin-bottom:20px; }
    .remember-row input { accent-color:var(--accent); width:16px; height:16px; }
    .remember-row label { font-size:13px; color:var(--text-muted); cursor:pointer; }
    .login-error { background:var(--danger-light); border:1px solid rgba(239,68,68,0.2); color:var(--danger); padding:10px 14px; border-radius:var(--radius); font-size:13px; margin-bottom:16px; display:none; }
    .login-error.show { display:block; animation:fadeIn 0.3s; }
    .btn { display:inline-flex; align-items:center; gap:7px; padding:10px 18px; border-radius:var(--radius); border:1px solid transparent; font-size:13px; font-weight:600; font-family:inherit; cursor:pointer; transition:all 0.2s; white-space:nowrap; }
    .btn:active { transform:scale(0.97); }
    .btn-blue { background:var(--accent); color:#fff; border-color:var(--accent); }
    .btn-blue:hover { background:var(--accent-hover); }
    .btn-full { width:100%; justify-content:center; padding:12px; font-size:14px; }
    .btn-outline { background:transparent; color:var(--text-muted); border-color:var(--border); }
    .btn-outline:hover { border-color:var(--accent); color:var(--accent); }
    .btn-ghost { background:var(--bg-elevated); color:var(--text-secondary); border-color:var(--border); }
    .btn-ghost:hover { background:var(--bg-hover); color:var(--text); border-color:var(--border-light); }
    .btn-success { background:var(--success); color:#fff; }
    .btn-success:hover { background:#16a34a; }
    .btn-warning { background:var(--warning); color:#000; }
    .btn-warning:hover { background:#d97706; }
    .btn-danger-outline { background:transparent; color:var(--danger); border-color:rgba(239,68,68,0.25); }
    .btn-danger-outline:hover { background:var(--danger-light); border-color:var(--danger); }
    .btn-sm { padding:6px 12px; font-size:12px; }
    .btn:disabled { opacity:0.5; cursor:not-allowed; transform:none; }
    .app { display:none; height:100vh; }
    .app.active { display:flex; }
    .sidebar { width:256px; background:var(--bg-card); border-right:1px solid var(--border); display:flex; flex-direction:column; flex-shrink:0; }
    .sidebar-header { padding:20px; border-bottom:1px solid var(--border); display:flex; align-items:center; gap:12px; }
    .sidebar-logo { width:36px; height:36px; background:linear-gradient(135deg,var(--accent),#6366f1); border-radius:10px; display:flex; align-items:center; justify-content:center; font-size:16px; color:white; }
    .sidebar-header .brand { font-weight:700; font-size:15px; }
    .sidebar-header .brand-sub { font-size:11px; color:var(--text-dim); }
    .sidebar-nav { flex:1; padding:16px; overflow-y:auto; }
    .nav-group { margin-bottom:24px; }
    .nav-label { font-size:11px; font-weight:600; color:var(--text-dim); text-transform:uppercase; letter-spacing:0.5px; padding:0 12px; margin-bottom:8px; }
    .nav-item { display:flex; align-items:center; gap:10px; padding:10px 12px; border-radius:var(--radius); color:var(--text-muted); cursor:pointer; font-size:13px; font-weight:500; transition:all 0.15s; border:1px solid transparent; }
    .nav-item:hover { background:var(--bg-elevated); color:var(--text); }
    .nav-item.active { background:var(--accent-light); color:var(--accent); border-color:var(--accent-glow); }
    .nav-item i { width:18px; font-size:14px; text-align:center; }
    .nav-badge { margin-left:auto; font-size:11px; font-weight:600; background:var(--accent); color:#fff; padding:2px 8px; border-radius:10px; }
    .sidebar-footer { padding:16px; border-top:1px solid var(--border); }
    .user-block { display:flex; align-items:center; gap:10px; padding:10px 12px; background:var(--bg); border-radius:var(--radius); margin-bottom:12px; }
    .user-avatar { width:32px; height:32px; background:var(--accent); border-radius:50%; display:flex; align-items:center; justify-content:center; font-weight:700; font-size:13px; color:white; }
    .user-name { font-size:13px; font-weight:600; }
    .user-role { font-size:11px; color:var(--text-muted); }
    .btn-logout { width:100%; padding:10px; background:transparent; border:1px solid var(--border); border-radius:var(--radius); color:var(--text-muted); font-size:13px; font-weight:500; cursor:pointer; display:flex; align-items:center; justify-content:center; gap:8px; transition:all 0.2s; font-family:inherit; }
    .btn-logout:hover { background:var(--danger-light); border-color:var(--danger); color:var(--danger); }
    .main { flex:1; display:flex; flex-direction:column; overflow:hidden; min-width:0; }
    .topbar { height:56px; display:flex; align-items:center; justify-content:space-between; padding:0 28px; border-bottom:1px solid var(--border); background:var(--bg-card); flex-shrink:0; }
    .topbar-title { font-size:18px; font-weight:700; display:flex; align-items:center; gap:10px; }
    .topbar-title i { color:var(--accent); }
    .topbar-actions { display:flex; gap:8px; }
    .content { flex:1; overflow-y:auto; overflow-x:hidden; }
    .view { display:none; padding:24px 28px 48px; }
    .view.active { display:block; }
    .date-nav { display:flex; align-items:center; gap:12px; margin-bottom:20px; background:var(--bg-card); border:1px solid var(--border); border-radius:var(--radius-lg); padding:14px 18px; }
    .date-nav-arrow { width:36px; height:36px; border-radius:var(--radius); background:var(--bg-elevated); border:1px solid var(--border); color:var(--text); cursor:pointer; font-size:13px; display:flex; align-items:center; justify-content:center; transition:all 0.15s; }
    .date-nav-arrow:hover { border-color:var(--accent); color:var(--accent); }
    .date-nav-arrow:active { transform:scale(0.92); }
    .date-nav-center { flex:1; text-align:center; }
    .date-nav-day { font-size:17px; font-weight:700; }
    .date-nav-sub { font-size:12px; color:var(--text-muted); margin-top:1px; }
    .date-nav input[type=date] { padding:8px 12px; background:var(--bg); border:1px solid var(--border); border-radius:var(--radius); color:var(--text); font-size:13px; cursor:pointer; font-family:inherit; }
    .date-nav input[type=date]:focus { outline:none; border-color:var(--accent); }
    .date-nav .today-pill { padding:8px 16px; border-radius:var(--radius); border:1px solid var(--accent); background:var(--accent-light); color:var(--accent); font-size:12px; font-weight:600; cursor:pointer; transition:all 0.15s; font-family:inherit; }
    .date-nav .today-pill:hover { background:var(--accent); color:#fff; }
    .date-nav .today-pill:active { transform:scale(0.95); }
    .kpi-row { display:grid; grid-template-columns:repeat(5,1fr); gap:14px; margin-bottom:22px; }
    .kpi-card { background:var(--bg-card); border:1px solid var(--border); border-radius:var(--radius-lg); padding:16px 18px; position:relative; overflow:hidden; transition:transform 0.25s cubic-bezier(0.16,1,0.3,1),box-shadow 0.25s; }
    .kpi-card:hover { transform:translateY(-2px); box-shadow:0 8px 24px rgba(0,0,0,0.3); }
    .kpi-card::before { content:''; position:absolute; top:0; left:0; width:4px; height:100%; }
    .kpi-card.c-blue::before { background:var(--accent); } .kpi-card.c-green::before { background:var(--success); }
    .kpi-card.c-yellow::before { background:var(--warning); } .kpi-card.c-red::before { background:var(--danger); }
    .kpi-card.c-purple::before { background:var(--purple); }
    .kpi-icon { width:30px; height:30px; border-radius:8px; display:flex; align-items:center; justify-content:center; font-size:13px; margin-bottom:10px; }
    .kpi-value { font-size:26px; font-weight:800; letter-spacing:-0.5px; margin-bottom:2px; }
    .kpi-label { font-size:12px; color:var(--text-muted); font-weight:500; }
    .filters { display:flex; align-items:center; gap:10px; flex-wrap:wrap; margin-bottom:16px; }
    .search-box { position:relative; flex:1; max-width:320px; }
    .search-box i { position:absolute; left:12px; top:50%; transform:translateY(-50%); color:var(--text-dim); font-size:13px; }
    .search-box input { width:100%; padding:10px 12px 10px 36px; background:var(--bg-card); border:1px solid var(--border); border-radius:var(--radius); color:var(--text); font-size:13px; font-family:inherit; outline:none; transition:border-color 0.2s; }
    .search-box input:focus { border-color:var(--accent); }
    .search-box input::placeholder { color:var(--text-dim); }
    .filter-select { padding:10px 12px; background:var(--bg-card); border:1px solid var(--border); border-radius:var(--radius); color:var(--text); font-size:13px; cursor:pointer; font-family:inherit; outline:none; transition:border-color 0.2s; }
    .filter-select:focus { border-color:var(--accent); }
    .filter-select option { background:var(--bg-card); }
    .mass-bar { display:none; align-items:center; gap:10px; flex-wrap:wrap; padding:12px 16px; margin-bottom:16px; background:var(--accent-light); border:1px solid var(--accent-glow); border-radius:var(--radius-lg); animation:fadeInUp 0.3s cubic-bezier(0.16,1,0.3,1); }
    .mass-bar.show { display:flex; }
    .mass-bar .mass-count { font-size:13px; font-weight:700; color:var(--accent); margin-right:6px; }
    .pkg-list { background:var(--bg-card); border:1px solid var(--border); border-radius:var(--radius-lg); overflow:hidden; }
    .pkg-row { display:flex; align-items:center; gap:14px; padding:14px 18px; border-bottom:1px solid var(--border); cursor:pointer; transition:background 0.15s; }
    .pkg-row:last-child { border-bottom:none; }
    .pkg-row:hover { background:var(--bg-hover); }
    .pkg-row:active { background:var(--bg-elevated); }
    .pkg-check { accent-color:var(--accent); width:16px; height:16px; cursor:pointer; flex-shrink:0; }
    .pkg-icon { width:38px; height:38px; border-radius:10px; display:flex; align-items:center; justify-content:center; font-size:15px; flex-shrink:0; }
    .pkg-body { flex:1; min-width:0; }
    .pkg-name { font-size:14px; font-weight:600; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
    .pkg-sub { font-size:12px; color:var(--text-dim); margin-top:1px; }
    .pkg-meta { display:flex; flex-direction:column; align-items:flex-end; gap:4px; flex-shrink:0; }
    .badge { display:inline-flex; align-items:center; gap:4px; padding:3px 10px; border-radius:6px; font-size:11px; font-weight:600; }
    .badge-blue { background:var(--accent-light); color:var(--accent); } .badge-green { background:var(--success-light); color:var(--success); }
    .badge-yellow { background:var(--warning-light); color:var(--warning); } .badge-red { background:var(--danger-light); color:var(--danger); }
    .badge-purple { background:var(--purple-light); color:var(--purple); } .badge-teal { background:var(--teal-light); color:var(--teal); }
    .badge-dim { background:var(--bg-elevated); color:var(--text-dim); }
    .pkg-chevron { color:var(--text-dim); font-size:12px; flex-shrink:0; margin-left:4px; }
    .detail-back { display:inline-flex; align-items:center; gap:6px; color:var(--accent); font-size:14px; font-weight:600; cursor:pointer; background:none; border:none; padding:0; margin-bottom:24px; font-family:inherit; transition:opacity 0.15s; }
    .detail-back:hover { opacity:0.7; }
    .detail-hero { display:flex; align-items:flex-start; justify-content:space-between; margin-bottom:28px; gap:16px; }
    .detail-id { font-size:12px; color:var(--text-dim); margin-bottom:4px; font-family:'SF Mono','Consolas',monospace; }
    .detail-title { font-size:26px; font-weight:800; letter-spacing:-0.5px; margin-bottom:10px; }
    .detail-badges { display:flex; gap:8px; flex-wrap:wrap; }
    .detail-actions { display:flex; gap:8px; flex-shrink:0; flex-wrap:wrap; }
    .info-grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(220px,1fr)); gap:14px; margin-bottom:28px; }
    .info-card { background:var(--bg-card); border:1px solid var(--border); border-radius:var(--radius-lg); padding:18px 20px; }
    .info-card-label { font-size:11px; font-weight:600; color:var(--text-dim); text-transform:uppercase; letter-spacing:0.5px; margin-bottom:6px; }
    .info-card-value { font-size:18px; font-weight:700; }
    .info-card-desc { font-size:12px; color:var(--text-muted); margin-top:4px; }
    .section-title { font-size:16px; font-weight:700; margin-bottom:14px; display:flex; align-items:center; gap:8px; }
    .section-title i { color:var(--accent); font-size:14px; }
    .quick-actions { display:flex; gap:10px; flex-wrap:wrap; margin-bottom:28px; }
    .timeline { padding-left:4px; }
    .tl-item { display:flex; gap:14px; padding-bottom:18px; position:relative; }
    .tl-item:last-child { padding-bottom:0; }
    .tl-dot { width:10px; height:10px; border-radius:50%; background:var(--accent); margin-top:5px; flex-shrink:0; position:relative; z-index:1; }
    .tl-item:not(:last-child) .tl-dot::after { content:''; position:absolute; left:4px; top:12px; width:2px; height:calc(100% + 8px); background:var(--border); }
    .tl-body { flex:1; }
    .tl-time { font-size:11px; color:var(--text-dim); }
    .tl-user { font-size:12px; color:var(--accent); font-weight:600; }
    .tl-desc { font-size:13px; color:var(--text-secondary); }
    .form-back { display:inline-flex; align-items:center; gap:6px; color:var(--accent); font-size:14px; font-weight:600; cursor:pointer; background:none; border:none; padding:0; margin-bottom:24px; font-family:inherit; }
    .form-back:hover { opacity:0.7; }
    .form-page-title { font-size:24px; font-weight:800; letter-spacing:-0.5px; margin-bottom:24px; }
    .form-card { background:var(--bg-card); border:1px solid var(--border); border-radius:var(--radius-lg); overflow:hidden; margin-bottom:18px; }
    .form-row { display:flex; align-items:center; padding:14px 20px; border-bottom:1px solid var(--border); gap:16px; }
    .form-row:last-child { border-bottom:none; }
    .form-row label { width:120px; flex-shrink:0; font-size:13px; font-weight:600; color:var(--text-secondary); }
    .form-row .input { flex:1; border:none; background:transparent; padding:0; font-size:14px; }
    .form-row .input:focus { box-shadow:none; }
    .form-row select.input { background:transparent; }
    .stat-hero-grid { display:grid; grid-template-columns:repeat(3,1fr); gap:14px; margin-bottom:28px; }
    .stat-hero-card { background:var(--bg-card); border:1px solid var(--border); border-radius:var(--radius-lg); padding:22px; text-align:center; transition:transform 0.25s cubic-bezier(0.16,1,0.3,1),box-shadow 0.25s; }
    .stat-hero-card:hover { transform:translateY(-2px); box-shadow:0 8px 24px rgba(0,0,0,0.3); }
    .stat-hero-icon { width:44px; height:44px; border-radius:12px; display:flex; align-items:center; justify-content:center; font-size:18px; margin:0 auto 12px; }
    .stat-hero-num { font-size:34px; font-weight:800; letter-spacing:-1px; }
    .stat-hero-desc { font-size:13px; color:var(--text-muted); margin-top:2px; }
    .stat-section { margin-bottom:28px; }
    .stat-section-title { font-size:16px; font-weight:700; margin-bottom:16px; display:flex; align-items:center; gap:8px; }
    .stat-section-title i { color:var(--accent); }
    .bar-row { display:flex; align-items:center; gap:14px; margin-bottom:12px; }
    .bar-label { width:140px; font-size:13px; font-weight:500; color:var(--text-secondary); flex-shrink:0; }
    .bar-track { flex:1; height:8px; background:var(--bg-elevated); border-radius:4px; overflow:hidden; }
    .bar-fill { height:100%; border-radius:4px; animation:barGrow 0.8s cubic-bezier(0.16,1,0.3,1) both; }
    .bar-val { width:54px; text-align:right; font-size:13px; font-weight:700; }
    .day-chart { display:flex; align-items:flex-end; gap:6px; height:110px; background:var(--bg-card); border:1px solid var(--border); border-radius:var(--radius-lg); padding:18px 16px 14px; }
    .day-bar-col { flex:1; display:flex; flex-direction:column; align-items:center; gap:4px; }
    .day-bar { width:100%; max-width:36px; border-radius:4px 4px 0 0; background:var(--accent); min-height:2px; transition:height 0.6s cubic-bezier(0.16,1,0.3,1); }
    .day-bar.today { background:var(--success); }
    .day-bar-lbl { font-size:10px; color:var(--text-dim); }
    .day-bar-cnt { font-size:11px; font-weight:700; }
    .user-row { display:flex; align-items:center; gap:14px; padding:14px 18px; border-bottom:1px solid var(--border); }
    .user-row:last-child { border-bottom:none; }
    .user-row-avatar { width:38px; height:38px; border-radius:50%; display:flex; align-items:center; justify-content:center; font-size:14px; font-weight:700; color:#fff; }
    .user-row-info { flex:1; }
    .user-row-name { font-size:14px; font-weight:600; }
    .user-row-sub { font-size:12px; color:var(--text-dim); }
    .sheet-overlay { position:fixed; inset:0; z-index:300; background:rgba(0,0,0,0.6); backdrop-filter:blur(6px); display:none; align-items:flex-end; justify-content:center; }
    .sheet-overlay.active { display:flex; }
    .sheet { width:100%; max-width:520px; background:var(--bg-card); border:1px solid var(--border); border-radius:16px 16px 0 0; padding:24px 24px 36px; animation:sheetUp 0.4s cubic-bezier(0.16,1,0.3,1); }
    .sheet-handle { width:36px; height:4px; background:var(--border-light); border-radius:2px; margin:0 auto 20px; }
    .sheet h3 { font-size:17px; font-weight:700; margin-bottom:16px; display:flex; align-items:center; gap:8px; }
    .sheet-note { font-size:13px; color:var(--warning); padding:10px 14px; background:var(--warning-light); border:1px solid rgba(245,158,11,0.2); border-radius:var(--radius); margin-bottom:16px; }
    .sheet-footer { display:flex; gap:10px; margin-top:20px; }
    .sheet-footer .btn { flex:1; justify-content:center; padding:12px; }
    .hist-row { display:flex; align-items:center; gap:14px; padding:12px 18px; border-bottom:1px solid var(--border); }
    .hist-row:last-child { border-bottom:none; }
    .hist-icon { width:34px; height:34px; border-radius:8px; display:flex; align-items:center; justify-content:center; font-size:13px; flex-shrink:0; }
    .hist-body { flex:1; min-width:0; }
    .hist-title { font-size:13px; font-weight:600; }
    .hist-sub { font-size:12px; color:var(--text-dim); }
    .hist-vals { display:flex; flex-direction:column; align-items:flex-end; gap:2px; flex-shrink:0; }
    .hist-old { font-size:11px; color:var(--text-dim); text-decoration:line-through; }
    .hist-new { font-size:11px; color:var(--success); font-weight:600; }
    .toast-wrap { position:fixed; top:16px; right:24px; z-index:9999; display:flex; flex-direction:column; gap:8px; }
    .toast { padding:12px 18px; border-radius:var(--radius); font-size:13px; font-weight:600; display:flex; align-items:center; gap:8px; animation:fadeInUp 0.35s cubic-bezier(0.16,1,0.3,1); box-shadow:0 8px 24px rgba(0,0,0,0.4); }
    .toast.success { background:#065f46; color:#a7f3d0; border:1px solid #059669; }
    .toast.error { background:#7f1d1d; color:#fca5a5; border:1px solid #dc2626; }
    .toast.info { background:#1e3a5f; color:#93c5fd; border:1px solid #3b82f6; }
    .empty-state { padding:60px 20px; text-align:center; color:var(--text-muted); }
    .empty-state i { font-size:44px; margin-bottom:14px; opacity:0.3; }
    .empty-state h3 { font-size:16px; font-weight:600; color:var(--text); margin-bottom:4px; }
    .loading-overlay { position:fixed; inset:0; background:var(--bg); z-index:500; display:flex; flex-direction:column; align-items:center; justify-content:center; gap:16px; }
    .spinner { width:36px; height:36px; border:3px solid var(--border); border-top-color:var(--accent); border-radius:50%; animation:spin 0.7s linear infinite; }
    @keyframes spin { to { transform:rotate(360deg); } }
    .loading-text { color:var(--text-muted); font-size:14px; }
    @media (max-width:960px) { .sidebar{display:none;} .kpi-row{grid-template-columns:repeat(2,1fr);} .stat-hero-grid{grid-template-columns:1fr;} .info-grid{grid-template-columns:1fr;} }
    @media (max-width:600px) { .kpi-row{grid-template-columns:1fr 1fr;} .filters{flex-direction:column;} .detail-hero{flex-direction:column;} }
  </style>
</head>
<body>

<!-- Loading -->
<div class="loading-overlay" id="loadingOv">
  <div class="spinner"></div>
  <div class="loading-text">Łączenie z bazą danych...</div>
</div>

<!-- LOGIN -->
<div class="login-screen" id="loginScreen" style="display:none;">
  <div class="login-card">
    <div class="login-header">
      <div class="login-icon"><i class="fa-solid fa-box-open"></i></div>
      <h1>Panel Wysyłek</h1>
      <p>Zaloguj się aby zarządzać paczkami</p>
    </div>
    <div class="login-error" id="loginError">Nieprawidłowy login lub hasło</div>
    <div class="field">
      <label>Login</label>
      <input class="input" type="text" id="loginUser" placeholder="Wpisz login" autocomplete="username">
    </div>
    <div class="field">
      <label>Hasło</label>
      <input class="input" type="password" id="loginPass" placeholder="Wpisz hasło" autocomplete="current-password">
    </div>
    <div class="remember-row">
      <input type="checkbox" id="loginRemember">
      <label for="loginRemember">Zapamiętaj mnie</label>
    </div>
    <button class="btn btn-blue btn-full" id="loginBtn" onclick="doLogin()">
      <i class="fa-solid fa-right-to-bracket"></i> Zaloguj się
    </button>
  </div>
</div>

<!-- APP -->
<div class="app" id="app">
  <aside class="sidebar">
    <div class="sidebar-header">
      <div class="sidebar-logo"><i class="fa-solid fa-truck-fast"></i></div>
      <div>
        <div class="brand">Wysyłki</div>
        <div class="brand-sub">Panel zarządzania</div>
      </div>
    </div>
    <nav class="sidebar-nav">
      <div class="nav-group">
        <div class="nav-label">Menu główne</div>
        <div class="nav-item active" id="navPkgs" onclick="go('pkgs')">
          <i class="fa-solid fa-boxes-stacked"></i> Paczki
          <span class="nav-badge" id="navBadge">0</span>
        </div>
        <div class="nav-item" id="navHist" onclick="go('hist')">
          <i class="fa-solid fa-clock-rotate-left"></i> Historia zmian
        </div>
        <div class="nav-item" id="navStats" onclick="go('stats')">
          <i class="fa-solid fa-chart-pie"></i> Statystyki
        </div>
      </div>
      <div class="nav-group" id="navAdminGrp">
        <div class="nav-label">Administracja</div>
        <div class="nav-item" id="navUsers" onclick="go('users')">
          <i class="fa-solid fa-users-gear"></i> Użytkownicy
        </div>
      </div>
    </nav>
    <div class="sidebar-footer">
      <div class="user-block">
        <div class="user-avatar" id="sAvatar">A</div>
        <div>
          <div class="user-name" id="sName">Admin</div>
          <div class="user-role" id="sRole">Administrator</div>
        </div>
      </div>
      <button class="btn-logout" onclick="doLogout()"><i class="fa-solid fa-arrow-right-from-bracket"></i> Wyloguj</button>
    </div>
  </aside>

  <div class="main">
    <div class="topbar">
      <div class="topbar-title" id="topTitle"><i class="fa-solid fa-boxes-stacked"></i> Paczki</div>
      <div class="topbar-actions" id="topActions"></div>
    </div>
    <div class="content" id="contentEl">

      <!-- PACKAGES VIEW -->
      <div class="view active" id="vPkgs">
        <div class="date-nav anim-up">
          <button class="date-nav-arrow" onclick="chgDate(-1)"><i class="fa-solid fa-chevron-left"></i></button>
          <div class="date-nav-center">
            <div class="date-nav-day" id="dDay">—</div>
            <div class="date-nav-sub" id="dSub">—</div>
          </div>
          <button class="date-nav-arrow" onclick="chgDate(1)"><i class="fa-solid fa-chevron-right"></i></button>
          <input type="date" id="dPick" onchange="goDate(this.value)">
          <button class="today-pill" onclick="goToday()">Dziś</button>
        </div>
        <div class="kpi-row" id="kpiRow"></div>
        <div class="filters">
          <div class="search-box">
            <i class="fa-solid fa-magnifying-glass"></i>
            <input type="text" placeholder="Szukaj po nazwie..." id="searchIn" oninput="renderList()">
          </div>
          <select class="filter-select" id="fCat" onchange="renderList()">
            <option value="">Wszystkie kategorie</option>
            <option value="Produkcja tył">Produkcja tył</option>
            <option value="Produkcja przód">Produkcja przód</option>
            <option value="Zewnętrzni dostawcy">Zewnętrzni dostawcy</option>
          </select>
          <select class="filter-select" id="fSt" onchange="renderList()">
            <option value="">Wszystkie statusy</option>
            <option value="W realizacji">W realizacji</option>
            <option value="Wysłano">Wysłano</option>
          </select>
          <select class="filter-select" id="fPr" onchange="renderList()">
            <option value="">Status druku</option>
            <option value="Nie wydrukowane">Nie wydrukowane</option>
            <option value="Wydrukowane">Wydrukowane</option>
          </select>
        </div>
        <div class="mass-bar" id="massBar">
          <span class="mass-count"><span id="massCnt">0</span> zaznaczonych</span>
          <button class="btn btn-success btn-sm" onclick="mSt('Wysłano')"><i class="fa-solid fa-check"></i> Wysłano</button>
          <button class="btn btn-ghost btn-sm" onclick="mSt('W realizacji')"><i class="fa-solid fa-rotate"></i> W realizacji</button>
          <button class="btn btn-warning btn-sm" onclick="mPr('Wydrukowane')"><i class="fa-solid fa-print"></i> Wydrukowane</button>
          <button class="btn btn-danger-outline btn-sm" onclick="mDel()"><i class="fa-solid fa-trash"></i> Usuń</button>
          <div style="flex:1"></div>
          <button class="btn btn-ghost btn-sm" onclick="clrSel()"><i class="fa-solid fa-xmark"></i> Odznacz</button>
        </div>
        <div id="pkgListEl"></div>
      </div>

      <!-- FORM VIEW -->
      <div class="view" id="vForm">
        <button class="form-back anim-fade" onclick="backToList()"><i class="fa-solid fa-chevron-left"></i> Powrót do listy</button>
        <div class="form-page-title anim-up" id="formTitle">Nowa paczka</div>
        <input type="hidden" id="fId">
        <div class="form-card anim-up d1">
          <div class="form-row"><label>Nazwa</label><input class="input" type="text" id="fNm" placeholder="Opis paczki..."></div>
          <div class="form-row"><label>Kategoria</label>
            <select class="input" id="fCatF">
              <option value="Produkcja tył">Produkcja tył</option>
              <option value="Produkcja przód">Produkcja przód</option>
              <option value="Zewnętrzni dostawcy">Zewnętrzni dostawcy</option>
            </select>
          </div>
        </div>
        <div class="form-card anim-up d2">
          <div class="form-row"><label>Realizacja</label>
            <select class="input" id="fStF">
              <option value="W realizacji">W realizacji</option>
              <option value="Wysłano">Wysłano</option>
            </select>
          </div>
          <div class="form-row"><label>Status druku</label>
            <select class="input" id="fPrF">
              <option value="Nie wydrukowane">Nie wydrukowane</option>
              <option value="Wydrukowane">Wydrukowane</option>
            </select>
          </div>
        </div>
        <div class="form-card anim-up d3">
          <div class="form-row"><label>Dzień wysyłki</label><input class="input" type="date" id="fDt"></div>
        </div>
        <div class="anim-up d4" style="margin-top:24px;display:flex;gap:10px;">
          <button class="btn btn-blue" id="saveBtn" onclick="saveForm()" style="padding:12px 28px;"><i class="fa-solid fa-floppy-disk"></i> Zapisz paczkę</button>
          <button class="btn btn-ghost" onclick="backToList()">Anuluj</button>
        </div>
      </div>

      <!-- DETAIL VIEW -->
      <div class="view" id="vDetail"></div>

      <!-- HISTORY VIEW -->
      <div class="view" id="vHist">
        <div class="section-title anim-up"><i class="fa-solid fa-clock-rotate-left"></i> Historia zmian</div>
        <div class="search-box anim-up d1" style="max-width:340px;margin-bottom:18px;">
          <i class="fa-solid fa-magnifying-glass"></i>
          <input type="text" placeholder="Szukaj w historii..." id="histSearch" oninput="renderHist()">
        </div>
        <div id="histListEl"></div>
      </div>

      <!-- STATS VIEW -->
      <div class="view" id="vStats"></div>

      <!-- USERS VIEW -->
      <div class="view" id="vUsers">
        <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:18px;" class="anim-up">
          <div class="section-title" style="margin-bottom:0;"><i class="fa-solid fa-users-gear"></i> Użytkownicy</div>
          <button class="btn btn-blue" onclick="openAddUser()"><i class="fa-solid fa-user-plus"></i> Nowy użytkownik</button>
        </div>
        <div class="pkg-list anim-up d1" id="usersListEl"></div>
      </div>

    </div>
  </div>
</div>

<!-- DELAY SHEET -->
<div class="sheet-overlay" id="delayOv" onclick="if(event.target===this)closeSheet('delayOv')">
  <div class="sheet">
    <div class="sheet-handle"></div>
    <h3><i class="fa-solid fa-clock" style="color:var(--warning)"></i> Opóźnij paczkę</h3>
    <div class="sheet-note"><i class="fa-solid fa-circle-info"></i> Opóźnienie liczone w <strong>dniach roboczych</strong> (pon–pt). Weekendy pomijane.</div>
    <input type="hidden" id="dlId">
    <div class="form-card">
      <div class="form-row"><label>Paczka</label><input class="input" type="text" id="dlName" readonly style="opacity:.6;"></div>
      <div class="form-row"><label>Obecna data</label><input class="input" type="text" id="dlCur" readonly style="opacity:.6;"></div>
      <div class="form-row"><label>Dni robocze</label><input class="input" type="number" id="dlDays" min="1" max="60" value="1" oninput="prevDelay()"></div>
      <div class="form-row"><label>Nowa data</label><input class="input" type="text" id="dlNew" readonly style="color:var(--accent);font-weight:700;"></div>
      <div class="form-row"><label>Powód</label><textarea class="input" id="dlReason" placeholder="Opisz powód opóźnienia..." rows="2"></textarea></div>
    </div>
    <div class="sheet-footer">
      <button class="btn btn-ghost" onclick="closeSheet('delayOv')">Anuluj</button>
      <button class="btn btn-warning" onclick="applyDelay()"><i class="fa-solid fa-clock"></i> Zastosuj opóźnienie</button>
    </div>
  </div>
</div>

<!-- ADD USER SHEET -->
<div class="sheet-overlay" id="userOv" onclick="if(event.target===this)closeSheet('userOv')">
  <div class="sheet">
    <div class="sheet-handle"></div>
    <h3><i class="fa-solid fa-user-plus" style="color:var(--accent)"></i> Nowy użytkownik</h3>
    <div class="form-card">
      <div class="form-row"><label>Login</label><input class="input" type="text" id="nuLogin" placeholder="login"></div>
      <div class="form-row"><label>Imię</label><input class="input" type="text" id="nuName" placeholder="Jan Kowalski"></div>
      <div class="form-row"><label>Hasło</label><input class="input" type="password" id="nuPass" placeholder="Silne hasło..."></div>
      <div class="form-row"><label>Rola</label>
        <select class="input" id="nuRole">
          <option value="user">Użytkownik</option>
          <option value="admin">Administrator</option>
        </select>
      </div>
    </div>
    <div class="sheet-footer">
      <button class="btn btn-ghost" onclick="closeSheet('userOv')">Anuluj</button>
      <button class="btn btn-blue" onclick="addUser()"><i class="fa-solid fa-user-plus"></i> Utwórz konto</button>
    </div>
  </div>
</div>

<div class="toast-wrap" id="toastWrap"></div>

<script>
// ============================================================
// KONFIGURACJA SUPABASE — wklej swoje dane tutaj
// ============================================================
const SUPABASE_URL = 'https://zqqbasuktqheztalqwkp.supabase.co';         // np. [xxxx.supabase.co](https://xxxx.supabase.co)
const SUPABASE_ANON = 'sb_publishable_FyUbpAW5jNK-6gh4YErLBw_sIT2e3d0';  // klucz anon/public
// ============================================================

const { createClient } = supabase;
const db = createClient(SUPABASE_URL, SUPABASE_ANON);

// ========== STATE ==========
const CATS = ['Produkcja tył', 'Produkcja przód', 'Zewnętrzni dostawcy'];
let user = null;
let curDate = new Date();
let pkgs = [], hist = [], users = [];
let sel = new Set();

// ========== BOOT ==========
(async function () {
  snapWD();
  updateDateUI();

  // Sprawdź zapisaną sesję
  const saved = localStorage.getItem('ship_s');
  if (saved) {
    try { user = JSON.parse(saved); } catch {}
  }

  // Załaduj dane z Supabase
  try {
    await Promise.all([loadPackages(), loadHistory(), loadUsers()]);
  } catch (e) {
    console.error('Błąd ładowania danych:', e);
  }

  document.getElementById('loadingOv').style.display = 'none';

  if (user) {
    enterApp();
  } else {
    document.getElementById('loginScreen').style.display = 'grid';
  }

  // Skróty klawiszowe logowania
  document.getElementById('loginPass').addEventListener('keydown', e => { if (e.key === 'Enter') doLogin(); });
  document.getElementById('loginUser').addEventListener('keydown', e => { if (e.key === 'Enter') document.getElementById('loginPass').focus(); });

  // Real-time — nasłuchuj zmian w tabelach
  db.channel('realtime-packages')
    .on('postgres_changes', { event: '*', schema: 'public', table: 'packages' }, () => {
      loadPackages().then(() => {
        if (curNav === 'pkgs') renderList();
        if (curNav === 'stats') renderStats();
      });
    })
    .subscribe();

  db.channel('realtime-history')
    .on('postgres_changes', { event: '*', schema: 'public', table: 'history' }, () => {
      loadHistory().then(() => {
        if (curNav === 'hist') renderHist();
      });
    })
    .subscribe();

  db.channel('realtime-users')
    .on('postgres_changes', { event: '*', schema: 'public', table: 'users' }, () => {
      loadUsers().then(() => {
        if (curNav === 'users') renderUsers();
      });
    })
    .subscribe();
})();

// ========== SUPABASE LOADERS ==========
async function loadPackages() {
  const { data, error } = await db.from('packages').select('*').order('ship_date');
  if (error) throw error;
  // Mapuj snake_case → camelCase
  pkgs = (data || []).map(p => ({
    id: p.id,
    name: p.name,
    category: p.category,
    status: p.status,
    printStatus: p.print_status,
    shipDate: p.ship_date,
    createdAt: p.created_at,
    delayed: p.delayed,
    delayReason: p.delay_reason,
    originalDate: p.original_date,
  }));
}

async function loadHistory() {
  const { data, error } = await db.from('history').select('*').order('created_at', { ascending: false }).limit(500);
  if (error) throw error;
  hist = (data || []).map(h => ({
    id: h.id,
    pid: h.package_id,
    pn: h.package_name,
    user: h.username,
    type: h.type,
    old: h.old_value,
    nw: h.new_value,
    ts: h.created_at,
  }));
}

async function loadUsers() {
  const { data, error } = await db.from('users').select('*').order('id');
  if (error) throw error;
  users = (data || []).map(u => ({
    id: u.id,
    login: u.login,
    name: u.name,
    password: u.password,
    role: u.role,
    created: u.created,
  }));
}

// ========== AUTH ==========
async function doLogin() {
  const l = document.getElementById('loginUser').value.trim();
  const p = document.getElementById('loginPass').value;
  const r = document.getElementById('loginRemember').checked;
  const btn = document.getElementById('loginBtn');

  btn.disabled = true;
  btn.innerHTML = '<i class="fa-solid fa-spinner fa-spin"></i> Logowanie...';

  // Pobierz użytkownika z bazy
  const { data, error } = await db.from('users').select('*').eq('login', l).eq('password', p).single();

  btn.disabled = false;
  btn.innerHTML = '<i class="fa-solid fa-right-to-bracket"></i> Zaloguj się';

  if (error || !data) {
    document.getElementById('loginError').classList.add('show');
    return;
  }

  document.getElementById('loginError').classList.remove('show');
  user = { login: data.login, name: data.name, role: data.role };
  if (r) localStorage.setItem('ship_s', JSON.stringify(user));
  enterApp();
}

function enterApp() {
  document.getElementById('loginScreen').style.display = 'none';
  document.getElementById('app').classList.add('active');
  document.getElementById('sName').textContent = user.name;
  document.getElementById('sRole').textContent = user.role === 'admin' ? 'Administrator' : 'Użytkownik';
  document.getElementById('sAvatar').textContent = user.name[0];
  document.getElementById('navAdminGrp').style.display = user.role === 'admin' ? '' : 'none';
  renderList();
}

function doLogout() {
  localStorage.removeItem('ship_s');
  user = null;
  document.getElementById('loginScreen').style.display = 'grid';
  document.getElementById('app').classList.remove('active');
  document.getElementById('loginUser').value = '';
  document.getElementById('loginPass').value = '';
}

// ========== NAV ==========
let curNav = 'pkgs';
function go(v) {
  curNav = v;
  document.querySelectorAll('.view').forEach(x => x.classList.remove('active'));
  document.querySelectorAll('.nav-item').forEach(x => x.classList.remove('active'));
  const ve = document.getElementById('v' + v.charAt(0).toUpperCase() + v.slice(1));
  if (ve) ve.classList.add('active');
  const ne = document.getElementById('nav' + v.charAt(0).toUpperCase() + v.slice(1));
  if (ne) ne.classList.add('active');
  const icons = { pkgs: 'fa-boxes-stacked', hist: 'fa-clock-rotate-left', stats: 'fa-chart-pie', users: 'fa-users-gear' };
  const titles = { pkgs: 'Paczki', hist: 'Historia zmian', stats: 'Statystyki', users: 'Użytkownicy' };
  document.getElementById('topTitle').innerHTML = `<i class="fa-solid ${icons[v] || ''}"></i> ${titles[v] || ''}`;
  setTopActions(v);
  if (v === 'stats') renderStats();
  if (v === 'hist') renderHist();
  if (v === 'users') renderUsers();
  if (v === 'pkgs') renderList();
  document.getElementById('contentEl').scrollTop = 0;
}
function setTopActions(v) {
  document.getElementById('topActions').innerHTML = v === 'pkgs'
    ? '<button class="btn btn-blue" onclick="openForm()"><i class="fa-solid fa-plus"></i> Nowa paczka</button>' : '';
}
function showView(id) {
  document.querySelectorAll('.view').forEach(x => x.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  document.getElementById('contentEl').scrollTop = 0;
}

// ========== DATE ==========
function isWE(d) { const w = d.getDay(); return w === 0 || w === 6; }
function addW(f, n) { const d = new Date(f), dir = n >= 0 ? 1 : -1; let r = Math.abs(n); while (r > 0) { d.setDate(d.getDate() + dir); if (!isWE(d)) r--; } return d; }
function snapWD() { while (isWE(curDate)) curDate.setDate(curDate.getDate() + 1); }
function fD(d) { return d.toISOString().split('T')[0]; }
function fDT(d) { return d.toISOString().replace('T', ' ').slice(0, 19); }
function fPL(s) {
  if (!s) return '—';
  const d = new Date(s + 'T00:00:00');
  const dn = ['Niedziela', 'Poniedziałek', 'Wtorek', 'Środa', 'Czwartek', 'Piątek', 'Sobota'];
  const mn = ['stycznia', 'lutego', 'marca', 'kwietnia', 'maja', 'czerwca', 'lipca', 'sierpnia', 'września', 'października', 'listopada', 'grudnia'];
  return `${dn[d.getDay()]}, ${d.getDate()} ${mn[d.getMonth()]} ${d.getFullYear()}`;
}
function chgDate(n) { curDate = addW(curDate, n); updateDateUI(); renderList(); }
function goDate(v) { if (!v) return; curDate = new Date(v + 'T00:00:00'); snapWD(); updateDateUI(); renderList(); }
function goToday() { curDate = new Date(); snapWD(); updateDateUI(); renderList(); }
function updateDateUI() {
  const ds = fD(curDate);
  document.getElementById('dDay').textContent = fPL(ds);
  document.getElementById('dSub').textContent = ds;
  document.getElementById('dPick').value = ds;
}

// ========== PACKAGE LIST ==========
function getF() {
  const ds = fD(curDate);
  const q = (document.getElementById('searchIn')?.value || '').trim().toLowerCase();
  const c = document.getElementById('fCat')?.value || '';
  const s = document.getElementById('fSt')?.value || '';
  const p = document.getElementById('fPr')?.value || '';
  let l = pkgs.filter(x => x.shipDate === ds);
  if (q) l = l.filter(x => x.name.toLowerCase().includes(q));
  if (c) l = l.filter(x => x.category === c);
  if (s) l = l.filter(x => x.status === s);
  if (p) l = l.filter(x => x.printStatus === p);
  return l;
}

function renderList() {
  const ds = fD(curDate), all = pkgs.filter(x => x.shipDate === ds), list = getF();
  const kd = [
    { icon: 'fa-boxes-stacked', c: 'c-blue', ic: 'var(--accent-light)', icc: 'var(--accent)', v: all.length, l: 'Wszystkie' },
    { icon: 'fa-rotate', c: 'c-yellow', ic: 'var(--warning-light)', icc: 'var(--warning)', v: all.filter(x => x.status === 'W realizacji').length, l: 'W realizacji' },
    { icon: 'fa-paper-plane', c: 'c-green', ic: 'var(--success-light)', icc: 'var(--success)', v: all.filter(x => x.status === 'Wysłano').length, l: 'Wysłano' },
    { icon: 'fa-print', c: 'c-red', ic: 'var(--danger-light)', icc: 'var(--danger)', v: all.filter(x => x.printStatus === 'Nie wydrukowane').length, l: 'Nie wydrukowane' },
    { icon: 'fa-clock', c: 'c-purple', ic: 'var(--purple-light)', icc: 'var(--purple)', v: all.filter(x => x.delayed).length, l: 'Opóźnione' },
  ];
  document.getElementById('kpiRow').innerHTML = kd.map((k, i) => `
    <div class="kpi-card ${k.c} anim-up d${i + 1}">
      <div class="kpi-icon" style="background:${k.ic};color:${k.icc};"><i class="fa-solid ${k.icon}"></i></div>
      <div class="kpi-value" style="color:${k.icc};">${k.v}</div>
      <div class="kpi-label">${k.l}</div>
    </div>`).join('');
  document.getElementById('navBadge').textContent = all.length;

  const el = document.getElementById('pkgListEl');
  if (list.length === 0) {
    el.innerHTML = `<div class="empty-state anim-up"><i class="fa-solid fa-box-open"></i><h3>Brak paczek na ten dzień</h3><p>Kliknij „Nowa paczka" aby dodać</p></div>`;
    return;
  }
  const catStyle = c => c === 'Produkcja tył' ? { bg: 'var(--purple-light)', fg: 'var(--purple)', ic: 'fa-industry' }
    : c === 'Produkcja przód' ? { bg: 'var(--teal-light)', fg: 'var(--teal)', ic: 'fa-gears' }
    : { bg: 'var(--warning-light)', fg: 'var(--warning)', ic: 'fa-truck' };

  el.innerHTML = `<div class="pkg-list">${list.map((p, i) => {
    const cs = catStyle(p.category), chk = sel.has(p.id) ? 'checked' : '';
    return `<div class="pkg-row anim-up" style="animation-delay:${Math.min(i * 0.03, 0.35)}s">
      <input type="checkbox" class="pkg-check" ${chk} onclick="event.stopPropagation();togSel(${p.id})">
      <div class="pkg-icon" style="background:${cs.bg};color:${cs.fg};"><i class="fa-solid ${cs.ic}"></i></div>
      <div class="pkg-body" onclick="openDetail(${p.id})">
        <div class="pkg-name">${esc(p.name)}</div>
        <div class="pkg-sub">${esc(p.category)}${p.delayed ? ' · <span style="color:var(--danger);">Opóźniona</span>' : ''}</div>
      </div>
      <div class="pkg-meta" onclick="openDetail(${p.id})">
        <span class="badge ${p.status === 'Wysłano' ? 'badge-green' : 'badge-blue'}">${p.status === 'Wysłano' ? '<i class="fa-solid fa-check"></i>' : ''} ${p.status}</span>
        <span style="font-size:11px;color:${p.printStatus === 'Wydrukowane' ? 'var(--success)' : 'var(--text-dim)'};">
          <i class="fa-solid fa-${p.printStatus === 'Wydrukowane' ? 'check' : 'xmark'}"></i> ${p.printStatus}
        </span>
      </div>
      <div class="pkg-chevron" onclick="openDetail(${p.id})"><i class="fa-solid fa-chevron-right"></i></div>
    </div>`;
  }).join('')}</div>`;
  updMass();
}

// ========== SELECTION ==========
function togSel(id) { sel.has(id) ? sel.delete(id) : sel.add(id); updMass(); }
function clrSel() { sel.clear(); renderList(); }
function updMass() {
  const b = document.getElementById('massBar');
  if (sel.size > 0) { b.classList.add('show'); document.getElementById('massCnt').textContent = sel.size; }
  else b.classList.remove('show');
}
async function mSt(v) {
  const ids = [...sel];
  for (const id of ids) {
    const p = pkgs.find(x => x.id === id);
    if (p && p.status !== v) {
      await db.from('packages').update({ status: v }).eq('id', id);
      await aH(p, 'Status realizacji', p.status, v);
    }
  }
  clrSel();
  await loadPackages();
  renderList();
  toast('success', 'Status zmieniony');
}
async function mPr(v) {
  const ids = [...sel];
  for (const id of ids) {
    const p = pkgs.find(x => x.id === id);
    if (p && p.printStatus !== v) {
      await db.from('packages').update({ print_status: v }).eq('id', id);
      await aH(p, 'Status druku', p.printStatus, v);
    }
  }
  clrSel();
  await loadPackages();
  renderList();
  toast('success', 'Status druku zmieniony');
}
async function mDel() {
  if (!confirm('Usunąć zaznaczone paczki?')) return;
  const ids = [...sel];
  await db.from('packages').delete().in('id', ids);
  clrSel();
  await loadPackages();
  renderList();
  toast('info', 'Usunięto');
}

// ========== DETAIL ==========
function openDetail(id) {
  const p = pkgs.find(x => x.id === id); if (!p) return;
  function catStyle(c) {
    return c === 'Produkcja tył' ? { bc: 'badge-purple', bg: 'var(--purple-light)', fg: 'var(--purple)' }
      : c === 'Produkcja przód' ? { bc: 'badge-teal', bg: 'var(--teal-light)', fg: 'var(--teal)' }
      : { bc: 'badge-yellow', bg: 'var(--warning-light)', fg: 'var(--warning)' };
  }
  const cs = catStyle(p.category);
  const entries = hist.filter(h => h.pid === id);
  const el = document.getElementById('vDetail');
  el.innerHTML = `
    <button class="detail-back anim-fade" onclick="backToList()"><i class="fa-solid fa-chevron-left"></i> Powrót do listy</button>
    <div class="detail-hero anim-up">
      <div>
        <div class="detail-id">#${p.id} · Utworzono ${esc(p.createdAt ? p.createdAt.slice(0,19).replace('T',' ') : '—')}</div>
        <div class="detail-title">${esc(p.name)}</div>
        <div class="detail-badges">
          <span class="badge ${cs.bc}">${esc(p.category)}</span>
          <span class="badge ${p.status === 'Wysłano' ? 'badge-green' : 'badge-blue'}">${esc(p.status)}</span>
          <span class="badge ${p.printStatus === 'Wydrukowane' ? 'badge-green' : 'badge-yellow'}">
            <i class="fa-solid fa-${p.printStatus === 'Wydrukowane' ? 'check' : 'xmark'}"></i> ${esc(p.printStatus)}
          </span>
          ${p.delayed ? '<span class="badge badge-red"><i class="fa-solid fa-clock"></i> Opóźniona</span>' : ''}
        </div>
      </div>
      <div class="detail-actions">
        <button class="btn btn-ghost" onclick="openForm(${p.id})"><i class="fa-solid fa-pen"></i> Edytuj</button>
        <button class="btn btn-warning btn-sm" onclick="openDelay(${p.id})"><i class="fa-solid fa-clock"></i> Opóźnij</button>
      </div>
    </div>
    <div class="info-grid">
      <div class="info-card anim-up d1">
        <div class="info-card-label">Dzień wysyłki</div>
        <div class="info-card-value">${fPL(p.shipDate)}</div>
        <div class="info-card-desc">${esc(p.shipDate)}</div>
      </div>
      <div class="info-card anim-up d2">
        <div class="info-card-label">Data utworzenia</div>
        <div class="info-card-value" style="font-size:15px;">${esc(p.createdAt ? p.createdAt.slice(0,19).replace('T',' ') : '—')}</div>
      </div>
      ${p.delayed ? `<div class="info-card anim-up d3" style="border-color:rgba(239,68,68,0.25);">
        <div class="info-card-label" style="color:var(--danger);">Opóźnienie</div>
        <div class="info-card-value" style="color:var(--danger);">Tak</div>
        <div class="info-card-desc">Oryginalna data: ${esc(p.originalDate || '—')}</div>
        <div class="info-card-desc">Powód: ${esc(p.delayReason || '—')}</div>
      </div>` : ''}
    </div>
    <div class="section-title anim-up d3"><i class="fa-solid fa-bolt"></i> Szybkie akcje</div>
    <div class="quick-actions anim-up d4">
      ${p.status === 'W realizacji'
        ? `<button class="btn btn-success" onclick="qSt(${p.id},'Wysłano')"><i class="fa-solid fa-check"></i> Oznacz jako wysłano</button>`
        : `<button class="btn btn-ghost" onclick="qSt(${p.id},'W realizacji')"><i class="fa-solid fa-rotate"></i> Wróć do realizacji</button>`}
      ${p.printStatus === 'Nie wydrukowane'
        ? `<button class="btn btn-warning" onclick="qPr(${p.id},'Wydrukowane')"><i class="fa-solid fa-print"></i> Oznacz jako wydrukowane</button>`
        : `<button class="btn btn-ghost" onclick="qPr(${p.id},'Nie wydrukowane')"><i class="fa-solid fa-print"></i> Cofnij druk</button>`}
      <button class="btn btn-danger-outline" onclick="delPkg(${p.id})"><i class="fa-solid fa-trash"></i> Usuń</button>
    </div>
    <div class="section-title anim-up d5"><i class="fa-solid fa-clock-rotate-left"></i> Historia zmian</div>
    <div class="timeline anim-up d6">
      ${entries.length === 0 ? '<p style="color:var(--text-dim);">Brak wpisów</p>' : ''}
      ${entries.map(h => `<div class="tl-item">
        <div class="tl-dot"></div>
        <div class="tl-body">
          <div class="tl-time">${esc(h.ts ? h.ts.slice(0,19).replace('T',' ') : '')}</div>
          <div class="tl-user">${esc(h.user)}</div>
          <div class="tl-desc"><strong>${esc(h.type)}</strong>: <span style="color:var(--text-dim);text-decoration:line-through;">${esc(h.old)}</span> → <span style="color:var(--success);">${esc(h.nw)}</span></div>
        </div>
      </div>`).join('')}
    </div>`;
  showView('vDetail');
  document.getElementById('topTitle').innerHTML = `<i class="fa-solid fa-box"></i> Paczka #${id}`;
  document.getElementById('topActions').innerHTML = '';
}

async function delPkg(id) {
  if (!confirm(`Usunąć paczkę #${id}?`)) return;
  await db.from('packages').delete().eq('id', id);
  await loadPackages();
  backToList();
  toast('info', 'Paczka usunięta');
}

function backToList() { showView('vPkgs'); go('pkgs'); }

async function qSt(id, v) {
  const p = pkgs.find(x => x.id === id); if (!p) return;
  await db.from('packages').update({ status: v }).eq('id', id);
  await aH(p, 'Status realizacji', p.status, v);
  await loadPackages();
  await loadHistory();
  openDetail(id);
  toast('success', 'Status zmieniony');
}
async function qPr(id, v) {
  const p = pkgs.find(x => x.id === id); if (!p) return;
  await db.from('packages').update({ print_status: v }).eq('id', id);
  await aH(p, 'Status druku', p.printStatus, v);
  await loadPackages();
  await loadHistory();
  openDetail(id);
  toast('success', 'Status druku zmieniony');
}

// ========== FORM ==========
function openForm(editId) {
  const isE = typeof editId === 'number';
  document.getElementById('formTitle').textContent = isE ? 'Edytuj paczkę' : 'Nowa paczka';
  document.getElementById('fId').value = isE ? editId : '';
  if (isE) {
    const p = pkgs.find(x => x.id === editId); if (!p) return;
    document.getElementById('fNm').value = p.name;
    document.getElementById('fCatF').value = p.category;
    document.getElementById('fStF').value = p.status;
    document.getElementById('fPrF').value = p.printStatus;
    document.getElementById('fDt').value = p.shipDate;
  } else {
    document.getElementById('fNm').value = '';
    document.getElementById('fCatF').value = CATS[0];
    document.getElementById('fStF').value = 'W realizacji';
    document.getElementById('fPrF').value = 'Nie wydrukowane';
    document.getElementById('fDt').value = fD(curDate);
  }
  showView('vForm');
  document.getElementById('topTitle').innerHTML = `<i class="fa-solid fa-${isE ? 'pen' : 'plus'}"></i> ${isE ? 'Edycja paczki' : 'Nowa paczka'}`;
  document.getElementById('topActions').innerHTML = '';
}

async function saveForm() {
  const id = document.getElementById('fId').value;
  const nm = document.getElementById('fNm').value.trim();
  const ct = document.getElementById('fCatF').value;
  const st = document.getElementById('fStF').value;
  const pr = document.getElementById('fPrF').value;
  const dt = document.getElementById('fDt').value;
  if (!nm) { toast('error', 'Wpisz nazwę paczki'); return; }
  if (!dt) { toast('error', 'Wybierz datę wysyłki'); return; }

  const btn = document.getElementById('saveBtn');
  btn.disabled = true;

  if (id) {
    const p = pkgs.find(x => x.id === parseInt(id)); if (!p) return;
    const updates = { name: nm, category: ct, status: st, print_status: pr, ship_date: dt };
    await db.from('packages').update(updates).eq('id', parseInt(id));
    if (p.status !== st) await aH(p, 'Status realizacji', p.status, st);
    if (p.printStatus !== pr) await aH(p, 'Status druku', p.printStatus, pr);
    toast('success', 'Paczka zaktualizowana');
  } else {
    const { data, error } = await db.from('packages').insert({
      name: nm, category: ct, status: st, print_status: pr, ship_date: dt,
      delayed: false
    }).select().single();
    if (!error && data) {
      await db.from('history').insert({
        package_id: data.id, package_name: nm,
        username: user.login, type: 'Utworzenie',
        old_value: '—', new_value: st
      });
    }
    toast('success', 'Paczka dodana');
  }

  btn.disabled = false;
  await loadPackages();
  await loadHistory();
  backToList();
}

// ========== DELAY ==========
function openDelay(id) {
  const p = pkgs.find(x => x.id === id); if (!p) return;
  document.getElementById('dlId').value = id;
  document.getElementById('dlName').value = `#${id} — ${p.name}`;
  document.getElementById('dlCur').value = p.shipDate;
  document.getElementById('dlDays').value = 1;
  document.getElementById('dlReason').value = '';
  prevDelay();
  document.getElementById('delayOv').classList.add('active');
}
function prevDelay() {
  const d = document.getElementById('dlCur').value;
  const n = parseInt(document.getElementById('dlDays').value) || 0;
  if (!d || n < 1) { document.getElementById('dlNew').value = '—'; return; }
  const nd = addW(new Date(d + 'T00:00:00'), n);
  document.getElementById('dlNew').value = fD(nd) + ' (' + fPL(fD(nd)) + ')';
}
async function applyDelay() {
  const id = parseInt(document.getElementById('dlId').value);
  const days = parseInt(document.getElementById('dlDays').value) || 0;
  const reason = document.getElementById('dlReason').value.trim();
  if (days < 1) { toast('error', 'Podaj liczbę dni'); return; }
  if (!reason) { toast('error', 'Podaj powód opóźnienia'); return; }
  const p = pkgs.find(x => x.id === id); if (!p) return;
  const old = p.shipDate;
  const nd = fD(addW(new Date(p.shipDate + 'T00:00:00'), days));
  const originalDate = p.originalDate || old;
  await db.from('packages').update({
    ship_date: nd, delayed: true, delay_reason: reason, original_date: originalDate
  }).eq('id', id);
  await aH(p, 'Opóźnienie', old, `${nd} (+${days} dni rob.) — ${reason}`);
  closeSheet('delayOv');
  await loadPackages();
  await loadHistory();
  if (document.getElementById('vDetail').classList.contains('active')) openDetail(id);
  else renderList();
  toast('info', `Paczka opóźniona do ${nd}`);
}
function closeSheet(id) { document.getElementById(id).classList.remove('active'); }

// ========== HISTORY ==========
async function aH(p, type, o, n) {
  await db.from('history').insert({
    package_id: p.id,
    package_name: p.name,
    username: user ? user.login : 'system',
    type: type,
    old_value: o,
    new_value: n
  });
}
function renderHist() {
  const q = (document.getElementById('histSearch')?.value || '').trim().toLowerCase();
  let list = hist;
  if (q) list = list.filter(h => h.pn.toLowerCase().includes(q) || h.user.includes(q) || String(h.pid).includes(q));
  const typeIcon = t => t.includes('Utworzenie') ? { bg: 'var(--accent-light)', fg: 'var(--accent)', i: 'fa-plus' }
    : t.includes('druku') ? { bg: 'var(--warning-light)', fg: 'var(--warning)', i: 'fa-print' }
    : t.includes('Opóźnienie') ? { bg: 'var(--danger-light)', fg: 'var(--danger)', i: 'fa-clock' }
    : { bg: 'var(--success-light)', fg: 'var(--success)', i: 'fa-arrow-right' };
  document.getElementById('histListEl').innerHTML = `<div class="pkg-list">${list.slice(0, 150).map((h, i) => {
    const ti = typeIcon(h.type);
    return `<div class="hist-row anim-up" style="animation-delay:${Math.min(i * 0.02, 0.5)}s">
      <div class="hist-icon" style="background:${ti.bg};color:${ti.fg};"><i class="fa-solid ${ti.i}"></i></div>
      <div class="hist-body">
        <div class="hist-title">${esc(h.type)} <span style="color:var(--text-dim);font-weight:400;">#${h.pid} ${esc(h.pn)}</span></div>
        <div class="hist-sub"><span style="color:var(--accent);">${esc(h.user)}</span> · ${esc(h.ts ? h.ts.slice(0,19).replace('T',' ') : '')}</div>
      </div>
      <div class="hist-vals"><span class="hist-old">${esc(h.old)}</span><span class="hist-new">${esc(h.nw)}</span></div>
    </div>`;
  }).join('')}</div>`;
}

// ========== STATS ==========
function renderStats() {
  const el = document.getElementById('vStats');
  const t = pkgs.length, sent = pkgs.filter(x => x.status === 'Wysłano').length,
    prog = pkgs.filter(x => x.status === 'W realizacji').length,
    printed = pkgs.filter(x => x.printStatus === 'Wydrukowane').length,
    delayed = pkgs.filter(x => x.delayed).length;
  const sp = t ? (sent / t * 100).toFixed(1) : 0;
  const pp = t ? (printed / t * 100).toFixed(1) : 0;
  const ip = t ? (prog / t * 100).toFixed(1) : 0;
  const cData = CATS.map(c => ({ name: c, cnt: pkgs.filter(x => x.category === c).length }));
  const cMax = Math.max(...cData.map(x => x.cnt), 1);
  const cClr = ['var(--purple)', 'var(--teal)', 'var(--warning)'];
  const days = [];
  const today = fD(new Date());
  for (let i = -7; i <= 5; i++) {
    const d = addW(new Date(), i), ds = fD(d);
    days.push({ date: ds, cnt: pkgs.filter(x => x.shipDate === ds).length, today: ds === today });
  }
  const mxD = Math.max(...days.map(x => x.cnt), 1);
  el.innerHTML = `
    <div class="section-title anim-up" style="font-size:24px;font-weight:800;letter-spacing:-0.5px;margin-bottom:24px;"><i class="fa-solid fa-chart-pie"></i> Statystyki</div>
    <div class="stat-hero-grid">
      <div class="stat-hero-card anim-up d1">
        <div class="stat-hero-icon" style="background:var(--accent-light);color:var(--accent);"><i class="fa-solid fa-boxes-stacked"></i></div>
        <div class="stat-hero-num" style="color:var(--accent);">${t}</div>
        <div class="stat-hero-desc">Wszystkie paczki</div>
      </div>
      <div class="stat-hero-card anim-up d2">
        <div class="stat-hero-icon" style="background:var(--success-light);color:var(--success);"><i class="fa-solid fa-paper-plane"></i></div>
        <div class="stat-hero-num" style="color:var(--success);">${sp}%</div>
        <div class="stat-hero-desc">Wysłano (${sent} z ${t})</div>
      </div>
      <div class="stat-hero-card anim-up d3">
        <div class="stat-hero-icon" style="background:var(--danger-light);color:var(--danger);"><i class="fa-solid fa-clock"></i></div>
        <div class="stat-hero-num" style="color:var(--danger);">${delayed}</div>
        <div class="stat-hero-desc">Opóźnionych paczek</div>
      </div>
    </div>
    <div class="stat-section anim-up d3">
      <div class="stat-section-title"><i class="fa-solid fa-chart-simple"></i> Postęp realizacji</div>
      <div class="bar-row"><div class="bar-label">Wysłano</div><div class="bar-track"><div class="bar-fill" style="width:${sp}%;background:var(--success);"></div></div><div class="bar-val" style="color:var(--success);">${sp}%</div></div>
      <div class="bar-row"><div class="bar-label">W realizacji</div><div class="bar-track"><div class="bar-fill" style="width:${ip}%;background:var(--accent);"></div></div><div class="bar-val" style="color:var(--accent);">${ip}%</div></div>
      <div class="bar-row"><div class="bar-label">Wydrukowane</div><div class="bar-track"><div class="bar-fill" style="width:${pp}%;background:var(--warning);"></div></div><div class="bar-val" style="color:var(--warning);">${pp}%</div></div>
    </div>
    <div class="stat-section anim-up d4">
      <div class="stat-section-title"><i class="fa-solid fa-layer-group"></i> Kategorie</div>
      ${cData.map((c, i) => `<div class="bar-row">
        <div class="bar-label">${esc(c.name)}</div>
        <div class="bar-track"><div class="bar-fill" style="width:${(c.cnt / cMax * 100).toFixed(1)}%;background:${cClr[i]};"></div></div>
        <div class="bar-val">${c.cnt}</div>
      </div>`).join('')}
    </div>
    <div class="stat-section anim-up d5">
      <div class="stat-section-title"><i class="fa-solid fa-calendar-days"></i> Paczki wg dnia wysyłki</div>
      <div class="day-chart">${days.map(d => `
        <div class="day-bar-col">
          <div class="day-bar-cnt">${d.cnt}</div>
          <div class="day-bar${d.today ? ' today' : ''}" style="height:${Math.max(d.cnt / mxD * 70, 3)}px;"></div>
          <div class="day-bar-lbl">${d.date.slice(5)}</div>
        </div>`).join('')}
      </div>
    </div>
    <div class="stat-section anim-up d6">
      <div class="stat-section-title"><i class="fa-solid fa-clock-rotate-left"></i> Ostatnia aktywność</div>
      <div class="pkg-list">${hist.slice(0, 6).map(h => {
        const ti = h.type.includes('Utworzenie') ? { bg: 'var(--accent-light)', fg: 'var(--accent)', i: 'fa-plus' }
          : h.type.includes('druku') ? { bg: 'var(--warning-light)', fg: 'var(--warning)', i: 'fa-print' }
          : { bg: 'var(--success-light)', fg: 'var(--success)', i: 'fa-arrow-right' };
        return `<div class="hist-row">
          <div class="hist-icon" style="background:${ti.bg};color:${ti.fg};"><i class="fa-solid ${ti.i}"></i></div>
          <div class="hist-body">
            <div class="hist-title" style="font-size:13px;">${esc(h.type)} #${h.pid}</div>
            <div class="hist-sub"><span style="color:var(--accent);">${esc(h.user)}</span> · ${esc(h.ts ? h.ts.slice(0,19).replace('T',' ') : '')}</div>
          </div>
        </div>`;
      }).join('')}</div>
    </div>`;
}

// ========== USERS ==========
function renderUsers() {
  const el = document.getElementById('usersListEl');
  const grads = ['linear-gradient(135deg,var(--accent),#6366f1)', 'linear-gradient(135deg,var(--teal),var(--success))', 'linear-gradient(135deg,var(--warning),var(--danger))'];
  el.innerHTML = users.map((u, i) => `
    <div class="user-row">
      <div class="user-row-avatar" style="background:${grads[i % grads.length]};">${u.name[0]}</div>
      <div class="user-row-info">
        <div class="user-row-name">${esc(u.name)}</div>
        <div class="user-row-sub">${esc(u.login)} · ${u.role === 'admin' ? 'Administrator' : 'Użytkownik'} · od ${esc(u.created)}</div>
      </div>
      <span class="badge ${u.role === 'admin' ? 'badge-red' : 'badge-blue'}">${u.role === 'admin' ? 'Admin' : 'User'}</span>
      ${u.id !== 1 ? `<button class="btn btn-danger-outline btn-sm" onclick="delUser(${u.id})"><i class="fa-solid fa-trash"></i></button>` : ''}
    </div>`).join('');
}
function openAddUser() {
  ['nuLogin', 'nuName', 'nuPass'].forEach(x => document.getElementById(x).value = '');
  document.getElementById('nuRole').value = 'user';
  document.getElementById('userOv').classList.add('active');
}
async function addUser() {
  const l = document.getElementById('nuLogin').value.trim();
  const n = document.getElementById('nuName').value.trim();
  const p = document.getElementById('nuPass').value;
  const r = document.getElementById('nuRole').value;
  if (!l || !n || !p) { toast('error', 'Wypełnij wszystkie pola'); return; }
  if (users.find(u => u.login === l)) { toast('error', 'Login już istnieje'); return; }
  const { error } = await db.from('users').insert({ login: l, name: n, password: p, role: r });
  if (error) { toast('error', 'Błąd tworzenia użytkownika'); return; }
  closeSheet('userOv');
  await loadUsers();
  renderUsers();
  toast('success', `Użytkownik "${l}" utworzony`);
}
async function delUser(id) {
  if (id === 1) { toast('error', 'Nie można usunąć głównego admina'); return; }
  if (!confirm('Usunąć użytkownika?')) return;
  await db.from('users').delete().eq('id', id);
  await loadUsers();
  renderUsers();
  toast('info', 'Usunięto');
}

// ========== TOAST ==========
function toast(t, m) {
  const i = t === 'success' ? 'fa-circle-check' : t === 'error' ? 'fa-circle-xmark' : 'fa-circle-info';
  const el = document.createElement('div');
  el.className = 'toast ' + t;
  el.innerHTML = `<i class="fa-solid ${i}"></i> ${esc(m)}`;
  document.getElementById('toastWrap').appendChild(el);
  setTimeout(() => { el.style.opacity = '0'; el.style.transition = 'opacity 0.3s'; setTimeout(() => el.remove(), 300); }, 2800);
}

function esc(s) { if (!s) return ''; const d = document.createElement('div'); d.appendChild(document.createTextNode(String(s))); return d.innerHTML; }
</script>
</body>
</html>
