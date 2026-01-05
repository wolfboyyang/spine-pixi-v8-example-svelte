<script lang="ts">
    import { Application, Assets, Rectangle } from "pixi.js";
    import { Spine } from "@esotericsoftware/spine-pixi-v8";
    import { onMount } from "svelte";

    let app: Application | null = $state(null);
    const totalBunnies = 10;

    class BunnyV8 {
        view;

        gravity = 0.75;

        speedX = Math.random() * 10;
        speedY = Math.random() * 10 - 5;

        positionX = 0;
        positionY = 0;

        bounds;

        constructor(bounds: Rectangle) {
            const spineboy = Spine.from({
                atlas: "spineboyAtlas",
                skeleton: "spineboyData",
                scale: 0.125,
            });

            // Set the default mix time to use when transitioning
            // from one animation to the next.
            spineboy.state.data.defaultMix = 0.2;

            // Center the spine object on screen.
            spineboy.x = window.innerWidth / 2;
            spineboy.y =
                window.innerHeight / 2 + spineboy.getBounds().height / 2;

            // Set animation "cape-follow-example" on track 0, looped.
            spineboy.state.setAnimation(0, "run", true);

            this.view = spineboy;
            this.bounds = bounds;
        }

        update() {
            let pX = this.positionX;
            let pY = this.positionY;

            pX += this.speedX;
            pY += this.speedY;
            this.speedY += this.gravity;

            if (pX > this.bounds.right) {
                this.speedX *= -1;
                pX = this.bounds.right;
            } else if (pX < this.bounds.left) {
                this.speedX *= -1;
                pX = this.bounds.left;
            }

            if (pY > this.bounds.bottom) {
                this.speedY *= -0.85;
                pY = this.bounds.bottom;
                if (Math.random() > 0.5) {
                    this.speedY -= Math.random() * 6;
                }
            } else if (pY < this.bounds.top) {
                this.speedY = 0;
                pY = this.bounds.top;
            }

            this.view.position.x = this.positionX = pX;
            this.view.position.y = this.positionY = pY;
        }

        reset() {
            this.positionX = 0;
            this.positionY = 0;
        }
    }

    onMount(async () => {
        app = new Application();
        await app.init({
            width: window.innerWidth,
            height: window.innerHeight,
            resolution: window.devicePixelRatio || 1,
            autoDensity: true,
            resizeTo: window,
            backgroundColor: 0x2c3e50,
            hello: true,
        });

        // Add pixi canvas element (app.view) to the document's body
        document.body.appendChild(app.canvas);

        // Pre-load the skeleton data and atlas. You can also load .json skeleton data.
        Assets.add({ alias: "spineboyData", src: "/assets/spineboy-pro.skel" });
        Assets.add({
            alias: "spineboyAtlas",
            src: "/assets/spineboy-pro.atlas",
        });
        await Assets.load(["spineboyData", "spineboyAtlas"]);

        const bounds = new Rectangle(
            0,
            0,
            window.innerWidth,
            window.innerHeight,
        );

        const bunnies: BunnyV8[] = [];
        const bunnyPool: BunnyV8[] = [];

        function addBunny() {
            const bunny = bunnyPool.pop() || new BunnyV8(bounds);

            bunny.reset();

            if (app) app.stage.addChild(bunny.view);
            bunnies.push(bunny);
        }
        for (let i = 0; i < totalBunnies; i++) {
            addBunny();
        }

        let pause = false;

        app.canvas.addEventListener("mousedown", () => {
            pause = !pause;
        });

        function renderUpdate() {
            if (!pause) {
                for (let i = 0; i < bunnies.length; i++) {
                    bunnies[i].update();
                }
            }

            // bunnies[0].view.visible = !bunnies[0].view.visible;

            if (app) {
                app.render();
                requestAnimationFrame(renderUpdate);
            }
        }

        renderUpdate();
    });

    onMount(() => {
        return () => {
            if (app) {
                document.body.removeChild(app.canvas);
                app.destroy();
                app = null;
                Assets.reset();
            }
        };
    });
</script>

<h1>Bunny mark</h1>
