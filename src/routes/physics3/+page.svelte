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
        Assets.add({
            alias: "snowglobeData",
            src: "/assets/snowglobe-pro.skel",
        });
        Assets.add({
            alias: "snowglobeAtlas",
            src: "/assets/snowglobe-pma.atlas",
        });
        await Assets.load(["snowglobeData", "snowglobeAtlas"]);

        // Create the spine display object
        const snowglobe = Spine.from({
            skeleton: "snowglobeData",
            atlas: "snowglobeAtlas",
            scale: 0.25,
        });

        // Center the spine object on screen.
        snowglobe.x = window.innerWidth / 2;
        snowglobe.y = window.innerHeight / 2 + snowglobe.getBounds().height / 4;

        // Set animation "FOA" on track 0, looped.
        snowglobe.state.setAnimation(0, "shake", true);

        // Add the display object to the stage.
        app.stage.addChild(snowglobe);
    });
</script>

<h1>Physics 3</h1>
