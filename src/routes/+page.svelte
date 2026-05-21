<script lang="ts">
  import { resolve } from '$app/paths';

  const projects = [
    {
      title: 'Flip Clock',
      href: resolve('/clock'),
      tag: 'UI Motion',
      summary: 'Đồng hồ số phong cách cổ điển với hiệu ứng lật trang (flip). Tập trung xử lý tính toán thời gian thực và tối ưu hóa hiệu ứng chuyển động mượt mà trong Svelte 5.'
    },
    {
      title: 'Animate Planet',
      href: resolve('/planet'),
      tag: 'UI Motion',
      summary: 'Mô phỏng hệ hành lang quỹ đạo bằng CSS Animation và SVG. Minh họa cách điều phối các chuyển động phức tạp và quản lý trạng thái đồng bộ giữa các đối tượng đồ họa.'
    },
    {
      title: 'Christmas Tree',
      href: resolve('/christmastree'),
      tag: 'UI Motion',
      summary: 'Ứng dụng diễn họa cây thông Noel bằng mã nhị phân và hiệu ứng ánh sáng. Showcase khả năng render dữ liệu động kết hợp với các hiệu ứng thị giác theo mùa.'
    },
    {
      title: 'Tym Animate',
      href: resolve('/tym'),
      tag: 'UI Motion',
      summary: 'Hiệu ứng trái tym chuyển động'
    },
    {
      title: 'Game Tetris',
      href: resolve('/tetris'),
      tag: 'Game',
      summary: 'Trò chơi xếp gạch kinh điển. Thách thức logic xử lý mảng hai chiều, tính toán va chạm và hệ thống tính điểm, lý tưởng để viết kịch bản kiểm thử E2E phức tạp.'
    },
    {
      title: 'Game Tictactoe',
      href: resolve('/tictactoe'),
      tag: 'Game',
      summary: 'Trò chơi Caro đối kháng với giao diện Neon hiện đại. Sử dụng để kiểm tra logic thắng/thua và các kịch bản tương tác người dùng cơ bản qua Playwright.'
    },
    {
      title: 'Game Platformer',
      href: resolve('/platformer'),
      tag: 'Game',
      summary: 'Game đi ải 2D với trọng lực và va chạm vật lý. Trọng tâm vào việc xử lý vòng lặp game (Game Loop), sự kiện bàn phím và điều hướng camera theo nhân vật.'
    },
    {
      title: 'Excel Web',
      href: resolve('/excel'),
      tag: 'App',
      summary: 'Bảng tính trực tuyến hỗ trợ công thức toán học và tham chiếu ô. Trình diễn kỹ thuật quản lý dữ liệu hiệu suất cao với SvelteMap và engine tính toán công thức động.'
    }
];
</script>



<section class="showcase-shell">
  <div class="hero-copy">
    <p class="eyebrow">CURATED PROJECTS</p>
    <h1>Hệ sinh thái giao diện thử nghiệm.</h1>
    <p class="lede">
      Khám phá bộ sưu tập các component và route demo. Mỗi thẻ là một lối vào trực tiếp, 
      giúp trải nghiệm UI, animation và các tính năng mới trong môi trường thực tế.
    </p>
  </div>

  <div class="project-grid">
    {#each projects as project, index (index)}
      <a class="project-card" href={project.href} style={`--card-index:${index};`}>
        <div class="card-inner">
          <div class="card-top">
            <span class="tag">{project.tag}</span>
            <div class="arrow-circle">
                <span class="arrow" aria-hidden="true">→</span>
            </div>
          </div>

          <div class="card-body">
            <h2>{project.title}</h2>
            <p>{project.summary}</p>
          </div>

          <div class="card-footer">
            <span class="action-text">Explore</span>
            <span class="path-text">{project.href}</span>
          </div>
        </div>
      </a>
    {/each}
  </div>
</section>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');
  .showcase-shell {
    position: relative;
    z-index: 1;
    width: min(1160px, calc(100% - 3rem));
    margin: 0 auto;
    padding: 6rem 0 8rem;
  }

  .hero-copy {
    max-width: 55rem;
    margin-bottom: 4rem;
    text-align: center;
    margin-left: auto;
    margin-right: auto;
  }

  .eyebrow {
    margin: 0 0 1rem;
    color: #4790ff;
    font-size: 0.85rem;
    font-weight: 700;
    letter-spacing: 0.25em;
    text-transform: uppercase;
  }

  h1 {
    margin: 0;
    color: #f7f9fc;
    font-size: clamp(2.5rem, 7vw, 4.8rem);
    line-height: 1.05;
    letter-spacing: -0.05em;
    font-weight: 800;
  }

  .lede {
    max-width: 42rem;
    margin: 1.5rem auto 0;
    color: #94a3b8;
    font-size: clamp(1.1rem, 2.2vw, 1.25rem);
    line-height: 1.7;
    font-weight: 400;
  }

  /* --- Project Grid --- */
  .project-grid {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 1.5rem;
  }

  /* --- Project Card --- */
  .project-card {
    display: block;
    text-decoration: none;
    color: inherit;
    border-radius: 2rem;
    position: relative;
    transform: translateY(calc(var(--card-index) * 8px));
    transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1), box-shadow 0.3s ease;
  }

  .project-card:hover {
    transform: translateY(-8px) scale(1.02);
  }

  .card-inner {
    display: grid;
    grid-template-rows: auto 1fr auto;
    gap: 2.5rem;
    min-height: 350px;
    padding: 2rem;
    background: rgba(255, 255, 255, 0.03);
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    border-radius: 2rem;
    border: 1px solid rgba(255, 255, 255, 0.06);
    position: relative;
    overflow: hidden;
  }

  .project-card:hover .card-inner {
    background: rgba(255, 255, 255, 0.05);
    border-color: rgba(71, 144, 255, 0.3);
    box-shadow: 
      0 20px 40px rgba(0, 0, 0, 0.3),
      0 0 20px rgba(71, 144, 255, 0.1);
  }

  /* --- Card Top --- */
  .card-top {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
  }

  .tag {
    display: inline-flex;
    align-items: center;
    padding: 0.5rem 1rem;
    border-radius: 999px;
    background: rgba(71, 144, 255, 0.1);
    color: #60a5fa;
    font-size: 0.75rem;
    font-weight: 700;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    border: 1px solid rgba(71, 144, 255, 0.2);
  }

  .arrow-circle {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 2.5rem;
    height: 2.5rem;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.05);
    border: 1px solid rgba(255, 255, 255, 0.1);
    transition: background 0.2s ease, transform 0.2s ease;
  }

  .project-card:hover .arrow-circle {
    background: #4790ff;
    transform: rotate(-45deg);
  }

  .arrow {
    font-size: 1.2rem;
    color: #f7f9fc;
  }

  /* --- Card Body --- */
  .card-body h2 {
    margin: 0 0 1rem;
    font-size: 2.2rem;
    line-height: 1.1;
    font-weight: 700;
    letter-spacing: -0.04em;
    color: #f1f5f9;
  }

  .card-body p {
    margin: 0;
    line-height: 1.6;
    font-size: 1.05rem;
    color: #cbd5e1;
  }

  /* --- Card Footer --- */
  .card-footer {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    padding-top: 1.5rem;
    border-top: 1px solid rgba(255, 255, 255, 0.06);
    font-size: 0.9rem;
    font-weight: 600;
  }

  .action-text {
    color: #4790ff;
  }

  .path-text {
    color: #64748b;
    font-family: 'Courier New', Courier, monospace;
    font-weight: 400;
  }

  /* --- Responsive Media Queries --- */
  @media (max-width: 1024px) {
    .project-grid {
      grid-template-columns: repeat(2, 1fr);
      gap: 1.25rem;
    }
    
    .project-card {
        transform: none;
    }
  }

  @media (max-width: 768px) {
    .showcase-shell {
      padding: 4rem 0 5rem;
    }

    .hero-copy {
        margin-bottom: 3rem;
    }

    .project-grid {
      grid-template-columns: 1fr;
    }

    .card-inner {
      min-height: auto;
      gap: 2rem;
      padding: 1.75rem;
    }

    .card-body h2 {
        font-size: 1.8rem;
    }
  }

  @media (max-width: 480px) {
    .showcase-shell {
      width: min(100% - 2rem, 1120px);
    }

    .card-footer {
      flex-direction: column;
      align-items: flex-start;
      gap: 0.5rem;
    }
  }

</style>
