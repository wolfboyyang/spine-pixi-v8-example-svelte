<script lang="ts">
    import {
        Application,
        Assets,
        Rectangle,
        type FederatedPointerEvent,
    } from "pixi.js";
    import { Spine, Vector2 } from "@esotericsoftware/spine-pixi-v8";
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

        // Set the default mix time to use when transitioning
        // from one animation to the next.
        spineboy.state.data.defaultMix = 0.2;

        // Set looping animations "idle" on track 0 and "aim" on track 1.
        spineboy.state.setAnimation(0, "idle", true);
        spineboy.state.setAnimation(1, "aim", true);

        // Center the Spine object on screen.
        spineboy.x = window.innerWidth / 2;
        spineboy.y = window.innerHeight / 2 + spineboy.getBounds().height / 2;

        // Add the display object to the stage.
        app.stage.addChild(spineboy);
        app.stage.hitArea = new Rectangle(
            0,
            0,
            app.canvas.width,
            app.canvas.height,
        );

        // Make the stage interactive and register pointer events
        app.stage.eventMode = "dynamic";
        let isDragging = false;

        app.stage.on("pointerdown", (e) => {
            isDragging = true;
            setBonePosition(e);
        });

        app.stage.on("globalpointermove", (e) => {
            if (isDragging) setBonePosition(e);
        });

        app.stage.on("pointerup", (e) => (isDragging = false));

        const setBonePosition = (e: FederatedPointerEvent) => {
            // Transform the mouse/touch coordinates to Spineboy's coordinate
            // system origin. `position` is then relative to Spineboy's root
            // bone.
            const position = new Vector2(
                e.global.x - spineboy.x,
                e.global.y - spineboy.y,
            );

            // Find the crosshair bone.
            const crosshairBone = spineboy.skeleton.findBone("crosshair");

            if (crosshairBone) {
                // Take the mouse position, which is relative to the root bone,
                // and transform it to the crosshair bone's parent root bone
                // coordinate system via `worldToLocal()`. `position` is relative
                // to the crosshair bone's parent bone after this
                crosshairBone.parent?.worldToLocal(position);

                // Set the crosshair bone's position to the mouse position
                crosshairBone.x = position.x;
                crosshairBone.y = position.y;
            }
        };
    });
</script>

<h1>SpineBoy with Pixi.js & SvelteKit</h1>
