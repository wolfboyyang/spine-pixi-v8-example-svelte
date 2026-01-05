<script lang="ts">
    import { Application, Assets, Container, Graphics, Text } from "pixi.js";
    import { Spine, Vector2 } from "@esotericsoftware/spine-pixi-v8";
    import { onMount } from "svelte";

    let app: Application | null = $state(null);

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
        Assets.add({
            alias: "girlData",
            src: "/assets/celestial-circus-pro.skel",
        });
        Assets.add({
            alias: "girlAtlas",
            src: "/assets/celestial-circus-pma.atlas",
        });
        await Assets.load(["girlData", "girlAtlas"]);

        // Create the spine display object
        const girl = Spine.from({
            skeleton: "girlData",
            atlas: "girlAtlas",
            scale: 0.2,
        });

        // Center the spine object on screen.
        girl.x = window.innerWidth / 2;
        girl.y = window.innerHeight / 2 + girl.getBounds().height / 4;

        // Set animation "eyeblink-long" on track 0, looped.
        girl.state.setAnimation(0, "eyeblink-long", true);

        // Add the display object to the stage.
        app.stage.addChild(girl);

        // Make the stage interactive and register pointer events
        app.stage.eventMode = "dynamic";
        app.stage.hitArea = app.screen;
        let isDragging = false;
        let lastX = -1,
            lastY = -1;

        app.stage.on("pointerdown", (e) => {
            isDragging = true;
            let mousePosition = new Vector2(e.data.global.x, e.data.global.y);
            lastX = mousePosition.x;
            lastY = mousePosition.y;
        });

        app.stage.on("pointermove", (e) => {
            if (isDragging) {
                let mousePosition = new Vector2(e.global.x, e.global.y);
                girl.x += mousePosition.x - lastX;
                girl.y += mousePosition.y - lastY;
                girl.skeleton.physicsTranslate(
                    mousePosition.x - lastX,
                    mousePosition.y - lastY,
                );
                lastX = mousePosition.x;
                lastY = mousePosition.y;
            }
        });

        const endDrag = () => (isDragging = false);
        app.stage.on("pointerup", endDrag);
        app.stage.on("pointerupoutside", endDrag);
        document.addEventListener("fullscreenchange", () => {
            endDrag();
        });

        const buttonContainer = new Container();
        buttonContainer.position.set(0, 0);

        const buttonBackground = new Graphics()
            .roundRect(0, 0, 140, 100, 5)
            .fill({ color: 0x000000, alpha: 0.7 });
        buttonContainer.addChild(buttonBackground);

        const fontStyle = {
            fill: 0xdddddd,
            fontFamily: "sans-serif",
            fontSize: 16,
        };

        // Create the text label for the heading
        const textHeading = new Text({
            text: "Drag anywhere",
            style: fontStyle,
        }); // Button text and color
        textHeading.position.set(15, 15); // Set the position of the text within the button
        buttonContainer.addChild(textHeading);

        // Create the text label for the FPS counter
        const textFps = new Text({ text: "0 fps", style: fontStyle });
        textFps.position.set(15, 40);
        buttonContainer.addChild(textFps);

        // Create the text label for the button toggle fullscreen
        const textButton = new Text({
            text: "Fullscreen",
            style: fontStyle,
        }); // Button text and color
        textButton.position.set(15, 65); // Set the position of the text within the button
        buttonContainer.addChild(textButton);

        // Add the button container to the stage
        app.stage.addChild(buttonContainer);

        buttonContainer.interactive = true;
        buttonContainer.eventMode = "static";

        let fsEnabled = false;
        buttonContainer.on("pointerdown", () => {
            if (fsEnabled) {
                document.exitFullscreen();
                textButton.text = "Fullscreen";
            } else {
                if (app) app.renderer.canvas.requestFullscreen();
                textButton.text = "Windowed";
            }
            fsEnabled = !fsEnabled;
        });

        // app.ticker.maxFPS = 60;
        app.ticker.add(() => {
            // Get the current FPS value from the ticker and round it
            if (app) {
                const fps = app.ticker.FPS.toFixed(0);

                // Update the text content
                textFps.text = `${fps} fps`;
            }
        });
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

<h1>Physics 2: drag to move around</h1>
