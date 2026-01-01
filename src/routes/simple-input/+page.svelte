<script lang="ts">
    import { Application, Assets } from "pixi.js";
    import { Spine } from "@esotericsoftware/spine-pixi-v8";
    import { onMount } from "svelte";

    const app = new Application();

    onMount(async () => {
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

        // Create the spine display object
        const spineboy = Spine.from({
            atlas: "spineboyAtlas",
            skeleton: "spineboyData",
            scale: 0.5,
        });

        // Set the default animation and the
        // default mix for transitioning between animations.
        spineboy.state.setAnimation(0, "hoverboard", true);
        spineboy.state.data.defaultMix = 0.2;

        // Center the spine object on screen.
        spineboy.x = window.innerWidth / 2;
        spineboy.y = window.innerHeight / 2 + spineboy.getBounds().height / 2;
        console.log("Initial position:", spineboy.x, spineboy.y);

        // Make it so that you can interact with Spineboy.
        // Handle the case that you click/tap the screen.
        spineboy.eventMode = "static";
        spineboy.on("pointerdown", onClick);

        // Add the display object to the stage.
        app.stage.addChild(spineboy);

        // Add variables for movement, speed.
        let moveLeft = false;
        let moveRight = false;
        const speed = 5;

        // Handle the case that the keyboard keys specified below are pressed.
        function onKeyDown(key: KeyboardEvent) {
            if (key.code === "ArrowLeft" || key.code === "KeyA") {
                moveLeft = true;
                spineboy.skeleton.scaleX = -1;
            } else if (key.code === "ArrowRight" || key.code === "KeyD") {
                moveRight = true;
                spineboy.skeleton.scaleX = 1;
            }
        }

        // Handle when the keys are released, if they were pressed.
        function onKeyUp(key: KeyboardEvent) {
            if (key.code === "ArrowLeft" || key.code === "KeyA") {
                moveLeft = false;
            } else if (key.code === "ArrowRight" || key.code === "KeyD") {
                moveRight = false;
            }
        }

        // Handle if you click/tap the screen.
        function onClick() {
            spineboy.state.setAnimation(1, "shoot", false);
        }

        // Add event listeners so that the window will correctly handle input.
        window.addEventListener("keydown", onKeyDown);
        window.addEventListener("keyup", onKeyUp);

        // Update the application to move Spineboy if input is detected.
        // app.ticker.maxFPS = 60;
        app.ticker.add(() => {
            if (moveLeft) {
                spineboy.x -= speed;
            }
            if (moveRight) {
                spineboy.x += speed;
            }
        });
    });
</script>

<h1>Simple Input: use A/D to move left/right, click to shoot</h1>
