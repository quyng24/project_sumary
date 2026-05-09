<svelte:options runes={true} />
<script lang="ts">
    import BackHome from '$lib/components/BackHome.svelte';
import { onMount } from 'svelte';

    // Svelte 5 options

    type ParticleConfig = {
        length: number;
        duration: number;
        velocity: number;
        effect: number;
        size: number;
    };

    const settings: { particles: ParticleConfig } = {
        particles: {
            length: 500,     // Tổng số hạt tối đa
            duration: 2,     // Thời gian sống của hạt (giây)
            velocity: 100,   // Tốc độ hạt
            effect: -0.75,   // Gia tốc (âm để hạt chậm dần)
            size: 30         // Kích thước hạt
        }
    };

    class Point {
        x: number;
        y: number;

        constructor(x = 0, y = 0) {
            this.x = x;
            this.y = y;
        }

        clone() {
            return new Point(this.x, this.y);
        }

        length(len?: number) {
            if (len === undefined) return Math.sqrt(this.x * this.x + this.y * this.y);
            this.normalize();
            this.x *= len;
            this.y *= len;
            return this;
        }

        normalize() {
            const currentLen = this.length() as number;
            if (currentLen === 0) return this;
            this.x /= currentLen;
            this.y /= currentLen;
            return this;
        }
    }

    class Particle {
        position = new Point();
        velocity = new Point();
        acceleration = new Point();
        age = 0;

        initialize(x: number, y: number, dx: number, dy: number) {
            this.position.x = x;
            this.position.y = y;
            this.velocity.x = dx;
            this.velocity.y = dy;
            this.acceleration.x = dx * settings.particles.effect;
            this.acceleration.y = dy * settings.particles.effect;
            this.age = 0;
        }

        update(deltaTime: number) {
            this.position.x += this.velocity.x * deltaTime;
            this.position.y += this.velocity.y * deltaTime;
            this.velocity.x += this.acceleration.x * deltaTime;
            this.velocity.y += this.acceleration.y * deltaTime;
            this.age += deltaTime;
        }

        draw(context: CanvasRenderingContext2D, image: HTMLCanvasElement) {
            const ease = (t: number) => --t * t * t + 1;
            const progress = this.age / settings.particles.duration;
            const size = image.width * ease(progress);
            
            context.globalAlpha = 1 - progress;
            context.drawImage(
                image,
                this.position.x - size / 2,
                this.position.y - size / 2,
                size,
                size
            );
        }
    }

    class ParticlePool {
        particles: Particle[];
        firstActive = 0;
        firstFree = 0;
        count = 0;

        constructor(length: number) {
            this.particles = Array.from({ length }, () => new Particle());
        }

        add(x: number, y: number, dx: number, dy: number) {
            this.particles[this.firstFree].initialize(x, y, dx, dy);
            this.firstFree = (this.firstFree + 1) % this.particles.length;
            if (this.firstActive === this.firstFree) {
                this.firstActive = (this.firstActive + 1) % this.particles.length;
            }
        }

        update(deltaTime: number) {
            let i = this.firstActive;
            while (i !== this.firstFree) {
                this.particles[i].update(deltaTime);
                if (this.particles[i].age >= settings.particles.duration) {
                    this.firstActive = (this.firstActive + 1) % this.particles.length;
                }
                i = (i + 1) % this.particles.length;
            }
        }

        draw(context: CanvasRenderingContext2D, image: HTMLCanvasElement) {
            let i = this.firstActive;
            while (i !== this.firstFree) {
                this.particles[i].draw(context, image);
                i = (i + 1) % this.particles.length;
            }
        }
    }

    let canvas = $state<HTMLCanvasElement>();

    function pointOnHeart(t: number) {
        return new Point(
            160 * Math.pow(Math.sin(t), 3),
            130 * Math.cos(t) - 50 * Math.cos(2 * t) - 20 * Math.cos(3 * t) - 10 * Math.cos(4 * t) + 25
        );
    }

    // Tối ưu: Dùng Offscreen Canvas thay vì DataURL Image
    function createHeartCanvas() {
        const heartCanvas = document.createElement('canvas');
        const context = heartCanvas.getContext('2d')!;
        const size = settings.particles.size;
        heartCanvas.width = size;
        heartCanvas.height = size;

        const to = (t: number) => {
            const p = pointOnHeart(t);
            return {
                x: size / 2 + (p.x * size) / 350,
                y: size / 2 - (p.y * size) / 350
            };
        };

        context.beginPath();
        for (let t = -Math.PI; t <= Math.PI; t += 0.01) {
            const p = to(t);
            if (t === -Math.PI) context.moveTo(p.x, p.y);
            else context.lineTo(p.x, p.y);
        }
        context.fillStyle = '#ea80b0';
        context.fill();
        return heartCanvas;
    }

    onMount(() => {
        if (!canvas) return;
        const context = canvas.getContext('2d')!;
        const pool = new ParticlePool(settings.particles.length);
        const heartImg = createHeartCanvas();
        const particleRate = settings.particles.length / settings.particles.duration;
        
        let time = performance.now() / 1000;
        let frameId: number;

        function render() {
            const now = performance.now() / 1000;
            const deltaTime = now - time;
            time = now;

            context.clearRect(0, 0, canvas!.width, canvas!.height);

            // Sinh hạt mới
            const amount = particleRate * deltaTime;
            for (let i = 0; i < amount; i++) {
                const t = Math.PI - 2 * Math.PI * Math.random();
                const pos = pointOnHeart(t);
                const dir = pos.clone().length(settings.particles.velocity);
                pool.add(
                    canvas!.width / 2 + pos.x,
                    canvas!.height / 2 - pos.y,
                    dir.x,
                    -dir.y
                );
            }

            pool.update(deltaTime);
            pool.draw(context, heartImg);
            frameId = requestAnimationFrame(render);
        }

        function resize() {
            canvas!.width = window.innerWidth;
            canvas!.height = window.innerHeight;
        }

        window.addEventListener('resize', resize);
        resize();
        render();

        return () => {
            cancelAnimationFrame(frameId);
            window.removeEventListener('resize', resize);
        };
    });
</script>

<div>
    <BackHome/>
    <canvas bind:this={canvas}></canvas>
</div>


<style>
    :global(html, body) {
        margin: 0;
        padding: 0;
        width: 100%;
        height: 100%;
        overflow: hidden;
        background: #000;
    }

    canvas {
        position: fixed;
        inset: 0;
        width: 100%;
        height: 100%;
    }
</style>