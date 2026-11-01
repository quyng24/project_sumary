<script lang="ts">
  import { resolve } from '$app/paths';

  interface Project {
    title: string;
    href: string;
    category: 'motion' | 'game' | 'app';
    tag: string;
    icon: string;
    gate: string;
    summary: string;
    highlights: string[];
  }

  const projects: Project[] = [
    {
      title: 'Flip Clock',
      href: resolve('/clock'),
      category: 'motion',
      tag: 'UI Motion',
      icon: '⏱️',
      gate: 'GATE 01',
      summary: 'Đồng hồ số lật phong cách Split-flap cổ điển. Xử lý tính toán thời gian thực và diễn hoạt 3D lật trang mượt mà trong Svelte 5.',
      highlights: ['Svelte 5 Runes', 'CSS 3D Motion', 'Realtime Timer']
    },
    {
      title: 'Animate Planet',
      href: resolve('/planet'),
      category: 'motion',
      tag: 'UI Motion',
      icon: '🪐',
      gate: 'GATE 02',
      summary: 'Mô phỏng hệ quỹ đạo vũ trụ bằng SVG và CSS Keyframes. Điều phối chuyển động đồng bộ nhịp nhàng giữa các hành tinh và vành đai.',
      highlights: ['SVG Vectors', 'Orbital Physics', 'Pure CSS Motion']
    },
    {
      title: 'Christmas Tree',
      href: resolve('/christmastree'),
      category: 'motion',
      tag: 'UI Motion',
      icon: '🎄',
      gate: 'GATE 03',
      summary: 'Cây thông Noel tương tác dựng từ ma trận mã nhị phân và hạt ánh sáng. Showcase khả năng render dữ liệu động kết hợp âm nhạc lễ hội.',
      highlights: ['Binary Matrix', 'Audio Sync', 'Luminous FX']
    },
    {
      title: 'Heart Particle',
      href: resolve('/tym'),
      category: 'motion',
      tag: 'UI Motion',
      icon: '💖',
      gate: 'GATE 04',
      summary: 'Hiệu ứng trái tim rực rỡ từ hàng trăm hạt particle toán học. Mô phỏng động học gia tốc và quỹ đạo cong trên HTML5 Canvas 2D 60fps.',
      highlights: ['Canvas 2D', 'Particle Pool', 'Physics Velocity']
    },
    {
      title: 'Game Tetris',
      href: resolve('/tetris'),
      category: 'game',
      tag: 'Mini Game',
      icon: '🧱',
      gate: 'GATE 05',
      summary: 'Trò chơi xếp gạch kinh điển với đầy đủ cơ chế xoay, va chạm, xóa hàng và tính điểm. Thử nghiệm logic mảng 2D và kịch bản Playwright E2E.',
      highlights: ['2D Grid Logic', 'Collision Detect', 'Score Engine']
    },
    {
      title: 'Game Tictactoe',
      href: resolve('/tictactoe'),
      category: 'game',
      tag: 'Mini Game',
      icon: '⚔️',
      gate: 'GATE 06',
      summary: 'Trò chơi Caro đối kháng phong cách Neon Cyberpunk hiện đại. Tích hợp thuật toán phán đoán nước đi thông minh và hiệu ứng âm thanh.',
      highlights: ['Neon Glow UI', 'Turn Algorithm', 'Cyberpunk Sound']
    },
    {
      title: 'Game Platformer',
      href: resolve('/platformer'),
      category: 'game',
      tag: 'Mini Game',
      icon: '🎮',
      gate: 'GATE 07',
      summary: 'Game phiêu lưu đi ải 2D với trọng lực, va chạm và chướng ngại vật. Tập trung xử lý Game Loop 60fps, sự kiện phím và camera bám đuổi.',
      highlights: ['Game Loop 60fps', 'Platformer Gravity', 'Camera Tracking']
    },
    {
      title: 'Excel Web',
      href: resolve('/excel'),
      category: 'app',
      tag: 'Web App',
      icon: '📊',
      gate: 'GATE 08',
      summary: 'Bảng tính trực tuyến mini hỗ trợ công thức toán học và tham chiếu ô linh hoạt. Trình diễn kỹ thuật quản lý state hiệu suất cao với SvelteMap.',
      highlights: ['SvelteMap State', 'Dynamic Formula', 'High Perf Grid']
    },
    {
      title: 'Chrome Dino',
      href: resolve('/dino'),
      category: 'game',
      tag: 'Mini Game',
      icon: '🦖',
      gate: 'GATE 09',
      summary: 'Tái hiện trò chơi Khủng Long T-Rex mất mạng kinh điển của Google Chrome với đầy đủ xương rồng, chim bay, chu kỳ ngày/đêm và âm thanh 8-bit.',
      highlights: ['Pixel Art Canvas', 'Day/Night Cycle', 'Web Audio 8-Bit']
    }
  ];

  // Bầu trời sao tĩnh với vị trí xác định - 0% CPU, 100% GPU compositing nhẹ
  const backgroundStars = [
    { top: '8%', left: '12%', size: '2px', opacity: 0.8, duration: '4.2s', delay: '0.3s', twinkle: true },
    { top: '14%', left: '78%', size: '1.5px', opacity: 0.6, duration: '5.1s', delay: '1.2s', twinkle: true },
    { top: '22%', left: '35%', size: '1px', opacity: 0.4, duration: '3.8s', delay: '2.5s', twinkle: false },
    { top: '28%', left: '92%', size: '2px', opacity: 0.85, duration: '6.0s', delay: '0.8s', twinkle: true },
    { top: '35%', left: '18%', size: '1.5px', opacity: 0.5, duration: '4.5s', delay: '3.1s', twinkle: false },
    { top: '42%', left: '64%', size: '2px', opacity: 0.75, duration: '5.5s', delay: '1.7s', twinkle: true },
    { top: '48%', left: '82%', size: '1px', opacity: 0.4, duration: '3.6s', delay: '2.9s', twinkle: false },
    { top: '55%', left: '8%', size: '2px', opacity: 0.7, duration: '4.8s', delay: '0.5s', twinkle: true },
    { top: '62%', left: '46%', size: '1.5px', opacity: 0.55, duration: '5.8s', delay: '3.4s', twinkle: false },
    { top: '68%', left: '88%', size: '2px', opacity: 0.75, duration: '4.1s', delay: '1.9s', twinkle: true },
    { top: '74%', left: '26%', size: '1px', opacity: 0.45, duration: '3.9s', delay: '2.1s', twinkle: false },
    { top: '81%', left: '72%', size: '2px', opacity: 0.65, duration: '5.2s', delay: '0.7s', twinkle: true },
    { top: '88%', left: '15%', size: '1.5px', opacity: 0.6, duration: '4.6s', delay: '3.6s', twinkle: true },
    { top: '92%', left: '58%', size: '1px', opacity: 0.35, duration: '4.0s', delay: '1.4s', twinkle: false },
    { top: '11%', left: '50%', size: '2.5px', opacity: 0.9, duration: '6.2s', delay: '2.3s', twinkle: true },
    { top: '25%', left: '6%', size: '1px', opacity: 0.4, duration: '3.5s', delay: '0.9s', twinkle: false },
    { top: '38%', left: '85%', size: '1.5px', opacity: 0.55, duration: '4.7s', delay: '2.8s', twinkle: true },
    { top: '52%', left: '28%', size: '2px', opacity: 0.65, duration: '5.4s', delay: '1.1s', twinkle: false },
    { top: '78%', left: '42%', size: '2px', opacity: 0.8, duration: '4.9s', delay: '0.4s', twinkle: true },
    { top: '85%', left: '80%', size: '1.5px', opacity: 0.5, duration: '5.7s', delay: '2.6s', twinkle: false }
  ];

  // Sao băng lướt nhẹ, rơi giãn cách êm ái, hướng chuẩn từ trên xuống góc dưới trái
  const meteors = [
    { top: '-4vh', left: '88vw', length: '120px', duration: '7.5s', delay: '0s' },
    { top: '6vh', left: '96vw', length: '145px', duration: '9.2s', delay: '3.2s' },
    { top: '-2vh', left: '68vw', length: '110px', duration: '8s', delay: '6.8s' },
    { top: '14vh', left: '92vw', length: '135px', duration: '10.5s', delay: '10.2s' },
    { top: '2vh', left: '80vw', length: '125px', duration: '8.6s', delay: '13.8s' }
  ];

  let activeCategory = $state<'all' | 'motion' | 'game' | 'app'>('all');

  const filteredProjects = $derived(
    activeCategory === 'all' 
      ? projects 
      : projects.filter(p => p.category === activeCategory)
  );

  // Dữ liệu đếm số lượng dự án tự động cập nhật linh hoạt từ mảng projects
  const totalProjects = $derived(projects.length);
  const formattedTotal = $derived(String(totalProjects).padStart(2, '0'));

  const categories = $derived([
    { id: 'all' as const, label: 'Tất cả dự án', count: totalProjects },
    { id: 'motion' as const, label: 'UI Motion & Visual', count: projects.filter(p => p.category === 'motion').length },
    { id: 'game' as const, label: 'Mini Game', count: projects.filter(p => p.category === 'game').length },
    { id: 'app' as const, label: 'Web App', count: projects.filter(p => p.category === 'app').length }
  ]);
</script>

<svelte:head>
  <title>Svelte Mini Projects Hub | Phòng Thí Nghiệm Giao Diện Sáng Tạo</title>
  <meta
    name="description"
    content="Không gian hội tụ các dự án nhỏ, component UI sáng tạo, trò chơi và ứng dụng web thử nghiệm xây dựng trên nền tảng Svelte 5."
  />
</svelte:head>

<!-- Nền vũ trụ tĩnh thanh lịch & sao băng rơi nhẹ (Không hiệu ứng đổi màu liên tục, không gây đơ/treo scroll) -->
<div class="cosmos-bg" aria-hidden="true">
  <!-- Bầu trời sao tĩnh & lấp lánh nhẹ -->
  <div class="starfield">
    {#each backgroundStars as star, i (i)}
      <span
        class="star"
        class:twinkle={star.twinkle}
        style="--top: {star.top}; --left: {star.left}; --size: {star.size}; --opacity: {star.opacity}; --dur: {star.duration}; --del: {star.delay};"
      ></span>
    {/each}
  </div>

  <!-- Vệt sao băng rơi thanh thoát (Đầu sao dẫn hướng, đuôi trải dài phía sau) -->
  <div class="meteor-layer">
    {#each meteors as meteor, i (i)}
      <div
        class="meteor"
        style="--m-top: {meteor.top}; --m-left: {meteor.left}; --m-len: {meteor.length}; --m-dur: {meteor.duration}; --m-del: {meteor.delay};"
      >
        <span class="meteor-trail"></span>
        <span class="meteor-head"></span>
      </div>
    {/each}
  </div>
</div>

<div class="app-layout">
  <!-- Top Navigation (Responsive Mobile & Desktop) -->
  <header class="navbar">
    <div class="nav-shell">
      <div class="nav-brand">
        <span class="brand-badge">LAB</span>
        <div class="brand-name-wrap">
          <span class="brand-full">SVELTE.COLLECTION</span>
          <span class="brand-short">SVELTE.LAB</span>
        </div>
      </div>

      <div class="nav-status">
        <span class="live-dot"></span>
        <span class="live-full">{totalProjects} EXPERIMENTS READY</span>
        <span class="live-short">{totalProjects} DEMOS</span>
      </div>

      <nav class="nav-links">
        <a href="https://github.com/quyng24" target="_blank" rel="noreferrer" class="github-link">
          <svg class="git-icon" viewBox="0 0 24 24" fill="currentColor" width="16" height="16">
            <path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"/>
          </svg>
          <span class="git-text">GitHub</span>
        </a>
      </nav>
    </div>
  </header>

  <!-- Main Showcase Container -->
  <main class="showcase-container">
    <!-- Hero Header: Giới thiệu tập hợp các dự án nhỏ -->
    <section class="hero-section">
      <div class="hero-pill">
        <span class="pill-dot"></span>
        <span class="pill-full">SVELTE 5 MINI-PROJECTS SHOWCASE</span>
        <span class="pill-short">SVELTE 5 MINI LAB</span>
      </div>

      <h1 class="hero-title">
        Nơi tập hợp các dự án nhỏ &
        <span class="gradient-text">Không gian thử nghiệm giao diện</span>
      </h1>

      <p class="hero-desc">
        Tuyển tập các dự án mini, demo chuyển động UI tương tác, trò chơi cổ điển và công cụ web thử nghiệm được xây dựng trên nền tảng <strong>Svelte 5</strong>. Mỗi ô cửa bên dưới là một lối vào trực tiếp để bạn trải nghiệm và khám phá.
      </p>

      <!-- Stats Grid (Tối ưu responsive 2x2 trên mobile) -->
      <div class="stats-bar">
        <div class="stat-item">
          <span class="stat-number">{formattedTotal}</span>
          <span class="stat-label">Dự án mini</span>
        </div>
        <div class="stat-divider"></div>
        <div class="stat-item">
          <span class="stat-number">Svelte 5</span>
          <span class="stat-label">Runes Reactivity</span>
        </div>
        <div class="stat-divider"></div>
        <div class="stat-item">
          <span class="stat-number">60 FPS</span>
          <span class="stat-label">Chuyển động mượt</span>
        </div>
        <div class="stat-divider"></div>
        <div class="stat-item">
          <span class="stat-number">0% Lag</span>
          <span class="stat-label">Tối ưu hiệu năng</span>
        </div>
      </div>

      <!-- Category Filter Tabs -->
      <div class="category-tabs" role="tablist" aria-label="Bộ lọc dự án">
        {#each categories as cat (cat.id)}
          <button
            type="button"
            class="tab-btn"
            class:active={activeCategory === cat.id}
            onclick={() => activeCategory = cat.id}
            role="tab"
            aria-selected={activeCategory === cat.id}
          >
            <span>{cat.label}</span>
            <span class="tab-count">{cat.count}</span>
          </button>
        {/each}
      </div>
    </section>

    <!-- Portal Grid: Cửa vào các trang dự án -->
    <section class="portals-section" aria-label="Danh sách cửa vào các dự án">
      <div class="portal-grid">
        {#each filteredProjects as project (project.title)}
          <a href={project.href} class="portal-card group">
            <div class="portal-inner">
              <!-- Header của cửa vào: Số hiệu cổng + Phân loại -->
              <div class="portal-header">
                <div class="gate-badge">
                  <span class="gate-id">{project.gate}</span>
                  <span class="gate-status-dot"></span>
                </div>
                <span class={`type-tag tag-${project.category}`}>{project.tag}</span>
              </div>

              <!-- Biểu tượng & Tiêu đề -->
              <div class="portal-body">
                <div class="icon-wrap">
                  <span class="project-icon" aria-hidden="true">{project.icon}</span>
                </div>
                <h2 class="project-title">{project.title}</h2>
                <p class="project-summary">{project.summary}</p>

                <!-- Tech highlights -->
                <div class="tech-chips">
                  {#each project.highlights as tag}
                    <span class="chip">{tag}</span>
                  {/each}
                </div>
              </div>

              <!-- Lối vào cửa: Action Bar -->
              <div class="portal-footer">
                <div class="route-box">
                  <span class="route-prefix">route:</span>
                  <span class="route-path">{project.href}</span>
                </div>
                <div class="enter-btn">
                  <span class="enter-text">Mở cổng</span>
                  <span class="enter-arrow" aria-hidden="true">→</span>
                </div>
              </div>
            </div>
          </a>
        {/each}
      </div>
    </section>
  </main>

  <!-- Footer -->
  <footer class="site-footer">
    <div class="footer-shell">
      <div class="footer-left">
        <div class="footer-badge">
          <span class="pulse-emerald"></span>
          <span>SYSTEM RUNNING • SVELTE 5</span>
        </div>
        <p class="footer-credit">Mini Projects Showcase Portfolio © 2024</p>
      </div>

      <div class="footer-center">
        <span class="stack-tag">SvelteKit 2</span>
        <span class="dot-sep">•</span>
        <span class="stack-tag">Vite 8</span>
        <span class="dot-sep">•</span>
        <span class="stack-tag">Canvas 2D</span>
        <span class="dot-sep">•</span>
        <span class="stack-tag">TypeScript</span>
      </div>

      <div class="footer-right">
        <a href="https://github.com/quyng24" target="_blank" rel="noreferrer">GitHub Profile</a>
        <a href={resolve('/clock')}>Dự án đầu tiên</a>
      </div>
    </div>
  </footer>
</div>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap');

  :global(body) {
    margin: 0;
    padding: 0;
    min-height: 100vh;
    background-color: #030712;
    color: #f1f5f9;
    font-family: 'Plus Jakarta Sans', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    -webkit-font-smoothing: antialiased;
    overflow-x: hidden;
  }

  /* ==========================================================================
     VŨ TRỤ TĨNH & SAO BĂNG RƠI NHẸ (0% CPU, scroll siêu mượt không lag)
     ========================================================================== */
  .cosmos-bg {
    position: fixed;
    inset: 0;
    pointer-events: none;
    z-index: 0;
    overflow: hidden;
    background: 
      radial-gradient(ellipse at 50% 0%, rgba(30, 58, 138, 0.15) 0%, transparent 60%),
      radial-gradient(circle at 10% 40%, rgba(15, 23, 42, 0.4) 0%, transparent 50%),
      linear-gradient(180deg, #030712 0%, #060b17 50%, #020408 100%);
  }

  /* Starfield */
  .starfield,
  .meteor-layer {
    position: absolute;
    inset: 0;
  }

  .star {
    position: absolute;
    top: var(--top);
    left: var(--left);
    width: var(--size);
    height: var(--size);
    border-radius: 50%;
    background-color: #ffffff;
    opacity: var(--opacity);
    box-shadow: 0 0 3px rgba(255, 255, 255, 0.6);
  }

  .star.twinkle {
    animation: star-shimmer var(--dur) ease-in-out var(--del) infinite;
  }

  @keyframes star-shimmer {
    0%, 100% {
      opacity: calc(var(--opacity) * 0.4);
      transform: scale(0.85);
    }
    50% {
      opacity: var(--opacity);
      transform: scale(1.2);
    }
  }

  /* Meteors (Sao băng rơi nhẹ, tự nhiên và ngắt quãng - hướng chuẩn 215° từ trên phải xuống dưới trái) */
  .meteor {
    position: absolute;
    top: var(--m-top);
    left: var(--m-left);
    width: var(--m-len);
    height: 2px;
    transform: rotate(215deg);
    transform-origin: center center;
    pointer-events: none;
    opacity: 0;
    animation: meteor-glide var(--m-dur) cubic-bezier(0.2, 0.9, 0.3, 1) var(--m-del) infinite;
    will-change: transform, opacity;
  }

  .meteor-trail {
    position: absolute;
    inset: 0;
    border-radius: 999px;
    background: linear-gradient(
      90deg,
      transparent 0%,
      rgba(96, 165, 250, 0.25) 30%,
      rgba(186, 230, 253, 0.8) 80%,
      #ffffff 100%
    );
  }

  .meteor-head {
    position: absolute;
    right: -1px;
    top: 50%;
    width: 4px;
    height: 4px;
    border-radius: 50%;
    background-color: #ffffff;
    transform: translateY(-50%);
    box-shadow: 
      0 0 6px 1px #ffffff,
      0 0 12px 2px rgba(96, 165, 250, 0.9);
  }

  @keyframes meteor-glide {
    0% {
      opacity: 0;
      transform: rotate(215deg) translateX(0) scale(0.6);
    }
    2% {
      opacity: 1;
      transform: rotate(215deg) translateX(120px) scale(1);
    }
    18% {
      opacity: 1;
      transform: rotate(215deg) translateX(950px) scale(1);
    }
    22% {
      opacity: 0;
      transform: rotate(215deg) translateX(1200px) scale(0.7);
    }
    100% {
      opacity: 0;
      transform: rotate(215deg) translateX(1200px) scale(0.7);
    }
  }

  /* ==========================================================================
     LAYOUT CHÍNH
     ========================================================================== */
  .app-layout {
    position: relative;
    z-index: 1;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }

  /* Navbar */
  .navbar {
    position: sticky;
    top: 0;
    z-index: 50;
    border-bottom: 1px solid rgba(255, 255, 255, 0.08);
    background: rgba(3, 7, 18, 0.85);
    backdrop-filter: blur(14px);
    -webkit-backdrop-filter: blur(14px);
  }

  .nav-shell {
    max-width: 1220px;
    margin: 0 auto;
    padding: 0.85rem 1.5rem;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    box-sizing: border-box;
  }

  .nav-brand {
    display: flex;
    align-items: center;
    gap: 0.65rem;
    flex-shrink: 0;
  }

  .brand-badge {
    padding: 0.2rem 0.55rem;
    border-radius: 6px;
    background: rgba(56, 189, 248, 0.12);
    border: 1px solid rgba(56, 189, 248, 0.3);
    color: #38bdf8;
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.7rem;
    font-weight: 700;
    letter-spacing: 0.05em;
  }

  .brand-name-wrap {
    font-weight: 800;
    font-size: 1.05rem;
    letter-spacing: -0.02em;
    color: #f8fafc;
  }

  .brand-short {
    display: none;
  }

  .nav-status {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.35rem 0.85rem;
    border-radius: 999px;
    background: rgba(16, 185, 129, 0.08);
    border: 1px solid rgba(16, 185, 129, 0.2);
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.7rem;
    font-weight: 600;
    color: #34d399;
    letter-spacing: 0.08em;
    white-space: nowrap;
  }

  .live-dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background-color: #10b981;
    box-shadow: 0 0 8px #10b981;
    flex-shrink: 0;
  }

  .live-short {
    display: none;
  }

  .nav-links {
    flex-shrink: 0;
  }

  .github-link {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.45rem 0.85rem;
    border-radius: 8px;
    background: rgba(255, 255, 255, 0.06);
    border: 1px solid rgba(255, 255, 255, 0.1);
    color: #cbd5e1;
    text-decoration: none;
    font-size: 0.85rem;
    font-weight: 600;
    white-space: nowrap;
    transition: all 0.2s ease;
  }

  .github-link:hover {
    background: rgba(255, 255, 255, 0.12);
    color: #ffffff;
    border-color: rgba(255, 255, 255, 0.2);
  }

  /* ==========================================================================
     HERO SECTION
     ========================================================================== */
  .showcase-container {
    max-width: 1220px;
    margin: 0 auto;
    padding: 4.5rem 1.5rem 6rem;
    width: 100%;
    box-sizing: border-box;
    overflow-x: clip;
  }

  .hero-section {
    text-align: center;
    max-width: 840px;
    margin: 0 auto 4rem;
  }

  .hero-pill {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.4rem 1rem;
    border-radius: 999px;
    background: rgba(59, 130, 246, 0.08);
    border: 1px solid rgba(59, 130, 246, 0.25);
    margin-bottom: 1.5rem;
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.75rem;
    font-weight: 700;
    letter-spacing: 0.15em;
    color: #60a5fa;
    max-width: 100%;
    box-sizing: border-box;
  }

  .pill-dot {
    width: 7px;
    height: 7px;
    border-radius: 50%;
    background-color: #38bdf8;
    box-shadow: 0 0 8px #38bdf8;
    flex-shrink: 0;
  }

  .pill-short {
    display: none;
  }

  .hero-title {
    margin: 0 0 1.25rem;
    font-size: clamp(2rem, 5vw, 3.6rem);
    font-weight: 800;
    line-height: 1.2;
    letter-spacing: -0.03em;
    color: #ffffff;
    text-wrap: balance;
  }

  .gradient-text {
    display: block;
    background: linear-gradient(135deg, #60a5fa 0%, #a78bfa 50%, #f472b6 100%);
    -webkit-background-clip: text;
    background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  .hero-desc {
    margin: 0 auto 2.5rem;
    font-size: clamp(1rem, 2vw, 1.15rem);
    line-height: 1.7;
    color: #94a3b8;
    max-width: 680px;
  }

  .hero-desc strong {
    color: #e2e8f0;
  }

  /* Stats Bar */
  .stats-bar {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0.85rem 1.75rem;
    border-radius: 1rem;
    background: rgba(15, 23, 42, 0.6);
    border: 1px solid rgba(255, 255, 255, 0.08);
    backdrop-filter: blur(8px);
    gap: 1.75rem;
    margin-bottom: 2.5rem;
  }

  .stat-item {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .stat-number {
    font-family: 'JetBrains Mono', monospace;
    font-size: 1.15rem;
    font-weight: 700;
    color: #f8fafc;
  }

  .stat-label {
    font-size: 0.75rem;
    color: #64748b;
    margin-top: 0.15rem;
  }

  .stat-divider {
    width: 1px;
    height: 28px;
    background: rgba(255, 255, 255, 0.1);
  }

  /* Category Filter Tabs */
  .category-tabs {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
    gap: 0.6rem;
  }

  .tab-btn {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem 1.1rem;
    border-radius: 999px;
    background: rgba(255, 255, 255, 0.04);
    border: 1px solid rgba(255, 255, 255, 0.08);
    color: #94a3b8;
    font-size: 0.85rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .tab-btn:hover {
    color: #f8fafc;
    background: rgba(255, 255, 255, 0.08);
    border-color: rgba(255, 255, 255, 0.15);
  }

  .tab-btn.active {
    background: rgba(59, 130, 246, 0.15);
    border-color: rgba(59, 130, 246, 0.4);
    color: #60a5fa;
  }

  .tab-count {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.72rem;
    padding: 0.1rem 0.45rem;
    border-radius: 999px;
    background: rgba(0, 0, 0, 0.3);
  }

  /* ==========================================================================
     PORTAL GRID: CỬA VÀO CÁC DỰ ÁN
     ========================================================================== */
  .portals-section {
    width: 100%;
  }

  .portal-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
    gap: 1.5rem;
  }

  /* Portal Card (Cánh Cổng) */
  .portal-card {
    display: block;
    text-decoration: none;
    color: inherit;
    border-radius: 1.25rem;
    background: rgba(15, 23, 42, 0.4);
    border: 1px solid rgba(255, 255, 255, 0.08);
    position: relative;
    overflow: hidden;
    transition: transform 0.25s cubic-bezier(0.2, 0.8, 0.2, 1),
                border-color 0.25s ease,
                box-shadow 0.25s ease,
                background-color 0.25s ease;
  }

  .portal-card:hover {
    transform: translateY(-5px);
    background: rgba(17, 24, 39, 0.65);
    border-color: rgba(96, 165, 250, 0.35);
    box-shadow: 
      0 16px 36px rgba(0, 0, 0, 0.4),
      0 0 20px rgba(56, 189, 248, 0.1);
  }

  .portal-inner {
    padding: 1.75rem;
    display: flex;
    flex-direction: column;
    min-height: 330px;
    box-sizing: border-box;
  }

  /* Portal Header */
  .portal-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 1.5rem;
  }

  .gate-badge {
    display: inline-flex;
    align-items: center;
    gap: 0.45rem;
    padding: 0.25rem 0.65rem;
    border-radius: 6px;
    background: rgba(255, 255, 255, 0.05);
    border: 1px solid rgba(255, 255, 255, 0.08);
  }

  .gate-id {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.72rem;
    font-weight: 700;
    letter-spacing: 0.1em;
    color: #cbd5e1;
  }

  .gate-status-dot {
    width: 5px;
    height: 5px;
    border-radius: 50%;
    background-color: #10b981;
    box-shadow: 0 0 6px #10b981;
  }

  .type-tag {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.7rem;
    font-weight: 700;
    letter-spacing: 0.08em;
    padding: 0.3rem 0.75rem;
    border-radius: 999px;
    text-transform: uppercase;
  }

  .tag-motion {
    background: rgba(56, 189, 248, 0.1);
    border: 1px solid rgba(56, 189, 248, 0.25);
    color: #38bdf8;
  }

  .tag-game {
    background: rgba(168, 85, 247, 0.1);
    border: 1px solid rgba(168, 85, 247, 0.25);
    color: #c084fc;
  }

  .tag-app {
    background: rgba(52, 211, 153, 0.1);
    border: 1px solid rgba(52, 211, 153, 0.25);
    color: #34d399;
  }

  /* Portal Body */
  .portal-body {
    flex: 1;
    margin-bottom: 1.5rem;
  }

  .icon-wrap {
    width: 3rem;
    height: 3rem;
    border-radius: 12px;
    background: rgba(255, 255, 255, 0.04);
    border: 1px solid rgba(255, 255, 255, 0.08);
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 1.15rem;
    transition: transform 0.2s ease, background 0.2s ease;
  }

  .portal-card:hover .icon-wrap {
    transform: scale(1.08);
    background: rgba(59, 130, 246, 0.1);
    border-color: rgba(59, 130, 246, 0.3);
  }

  .project-icon {
    font-size: 1.5rem;
    line-height: 1;
  }

  .project-title {
    margin: 0 0 0.65rem;
    font-size: 1.45rem;
    font-weight: 700;
    letter-spacing: -0.02em;
    color: #f8fafc;
    transition: color 0.2s ease;
  }

  .portal-card:hover .project-title {
    color: #60a5fa;
  }

  .project-summary {
    margin: 0 0 1.25rem;
    font-size: 0.92rem;
    line-height: 1.6;
    color: #94a3b8;
  }

  /* Tech Chips */
  .tech-chips {
    display: flex;
    flex-wrap: wrap;
    gap: 0.4rem;
  }

  .chip {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.68rem;
    padding: 0.2rem 0.55rem;
    border-radius: 4px;
    background: rgba(255, 255, 255, 0.04);
    border: 1px solid rgba(255, 255, 255, 0.06);
    color: #94a3b8;
  }

  /* Portal Footer (Entrance Action) */
  .portal-footer {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    padding-top: 1.2rem;
    border-top: 1px solid rgba(255, 255, 255, 0.06);
  }

  .route-box {
    display: flex;
    align-items: center;
    gap: 0.35rem;
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.78rem;
  }

  .route-prefix {
    color: #475569;
  }

  .route-path {
    color: #64748b;
    transition: color 0.2s ease;
  }

  .portal-card:hover .route-path {
    color: #93c5fd;
  }

  .enter-btn {
    display: inline-flex;
    align-items: center;
    gap: 0.4rem;
    color: #60a5fa;
    font-size: 0.85rem;
    font-weight: 700;
    transition: color 0.2s ease;
  }

  .enter-arrow {
    font-size: 1.1rem;
    transition: transform 0.2s ease;
  }

  .portal-card:hover .enter-arrow {
    transform: translateX(4px);
  }

  /* ==========================================================================
     FOOTER
     ========================================================================== */
  .site-footer {
    margin-top: auto;
    border-top: 1px solid rgba(255, 255, 255, 0.08);
    background: rgba(3, 7, 18, 0.65);
    backdrop-filter: blur(12px);
    padding: 3rem 0;
  }

  .footer-shell {
    max-width: 1220px;
    margin: 0 auto;
    padding: 0 1.5rem;
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 1.5rem;
  }

  .footer-left {
    display: flex;
    flex-direction: column;
    gap: 0.4rem;
  }

  .footer-badge {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.7rem;
    color: #10b981;
    font-weight: 600;
    letter-spacing: 0.05em;
  }

  .pulse-emerald {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background-color: #10b981;
    box-shadow: 0 0 8px #10b981;
  }

  .footer-credit {
    margin: 0;
    font-size: 0.8rem;
    color: #64748b;
  }

  .footer-center {
    display: flex;
    align-items: center;
    gap: 0.65rem;
  }

  .stack-tag {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.72rem;
    color: #64748b;
  }

  .dot-sep {
    color: #334155;
    font-size: 0.75rem;
  }

  .footer-right {
    display: flex;
    align-items: center;
    gap: 1.5rem;
  }

  .footer-right a {
    color: #94a3b8;
    text-decoration: none;
    font-size: 0.85rem;
    font-weight: 600;
    transition: color 0.2s ease;
  }

  .footer-right a:hover {
    color: #38bdf8;
  }

  /* ==========================================================================
     RESPONSIVE
     ========================================================================== */
  @media (max-width: 1024px) {
    .portal-grid {
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: 1.25rem;
    }
  }

  @media (max-width: 768px) {
    .brand-full {
      display: none;
    }

    .brand-short {
      display: inline;
      font-size: 0.95rem;
    }

    .live-full {
      display: none;
    }

    .live-short {
      display: inline;
    }

    .pill-full {
      display: none;
    }

    .pill-short {
      display: inline;
    }

    .nav-shell {
      padding: 0.75rem 1rem;
    }

    .showcase-container {
      padding: 3rem 1rem 5rem;
    }

    .hero-section {
      margin-bottom: 2.75rem;
    }

    .hero-title {
      font-size: clamp(1.75rem, 6.5vw, 2.5rem);
    }

    .stats-bar {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 0.85rem;
      width: 100%;
      max-width: 440px;
      margin: 0 auto 2rem;
      padding: 1.15rem 1rem;
      box-sizing: border-box;
    }

    .stat-divider {
      display: none;
    }

    .portal-grid {
      grid-template-columns: 1fr;
    }

    .footer-shell {
      flex-direction: column;
      align-items: flex-start;
      gap: 1.5rem;
    }
  }

  @media (max-width: 480px) {
    .nav-status {
      display: none;
    }

    .nav-shell {
      padding: 0.65rem 0.85rem;
    }

    .github-link {
      padding: 0.35rem 0.65rem;
      font-size: 0.8rem;
    }

    .showcase-container {
      padding: 2rem 0.85rem 4rem;
    }

    .hero-section {
      margin-bottom: 2.25rem;
    }

    .hero-pill {
      padding: 0.35rem 0.75rem;
      letter-spacing: 0.08em;
    }

    .hero-desc {
      font-size: 0.92rem;
      line-height: 1.6;
      margin-bottom: 1.75rem;
    }

    .portal-inner {
      padding: 1.25rem;
      min-height: auto;
    }

    .project-title {
      font-size: 1.25rem;
    }

    .category-tabs {
      gap: 0.35rem;
    }

    .tab-btn {
      padding: 0.35rem 0.7rem;
      font-size: 0.76rem;
    }

    .meteor:nth-child(n + 4) {
      display: none;
    }
  }

  @media (prefers-reduced-motion: reduce) {
    .star.twinkle,
    .meteor {
      animation: none !important;
    }
    .meteor {
      display: none;
    }
  }
</style>
