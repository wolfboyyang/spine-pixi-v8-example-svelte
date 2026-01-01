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
            alias: "spineboyData",
            src: "/assets/dragon-ess.skel",
        });
        Assets.add({
            alias: "spineboyAtlas",
            src: "/assets/dragon-ess.atlas",
        });
        Assets.add({
            alias: "spineboyData2",
            src: "/assets/dragon-ess.skel",
        });
        Assets.add({
            alias: "spineboyAtlas2",
            src: "/assets/dragon-ess.atlas",
        });
        await Assets.load([
            "spineboyData",
            "spineboyAtlas",
            "spineboyData",
            "spineboyAtlas2",
        ]);

        // Create the spine display object
        const spineboy = Spine.from({
            skeleton: "spineboyData",
            atlas: "spineboyAtlas",
            scale: 0.5,
        });
        const spineboy2 = Spine.from({
            skeleton: "spineboyData",
            atlas: "spineboyAtlas",
            scale: 0.5,
        });
        spineboy.autoUpdate = false;
        spineboy2.autoUpdate = false;

        // Set the default mix time to use when transitioning
        // from one animation to the next.
        spineboy.state.data.defaultMix = 0.2;
        spineboy2.state.data.defaultMix = 0.2;

        // Center the spine object on screen.
        spineboy.x = window.innerWidth / 2;
        spineboy.y = window.innerHeight / 2 - 30;

        spineboy2.x = window.innerWidth / 2;
        spineboy2.y = window.innerHeight / 2 + 200;

        // Set animation "run" on track 0, looped.
        spineboy.state.setAnimation(0, "flying", true);
        spineboy2.state.setAnimation(0, "flying", true);

        // Add the display object to the stage.
        app.stage.addChild(spineboy);
        app.stage.addChild(spineboy2);

        app.ticker.add((_delta) => {
            spineboy.update(_delta.deltaTime / 100);
            spineboy2.update(_delta.deltaTime / 100);
        });
    });
</script>

<h1>Dragon</h1>
