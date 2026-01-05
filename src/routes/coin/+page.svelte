<script lang="ts">
    import { Application, Assets } from "pixi.js";
    import { Spine } from "@esotericsoftware/spine-pixi-v8";
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
        Assets.add({ alias: "spineboyData", src: "/assets/coin-pro.skel" });
        Assets.add({
            alias: "spineboyAtlas",
            src: "/assets/coin-pma.atlas",
        });
        await Assets.load(["spineboyData", "spineboyAtlas"]);

        // Create the spine display object
        const spineboy = Spine.from({
            skeleton: "spineboyData",
            atlas: "spineboyAtlas",
            scale: 1,
            // darkTint: true,
        });
        spineboy.autoUpdate = false;

        // Set the default mix time to use when transitioning
        // from one animation to the next.
        spineboy.state.data.defaultMix = 0.2;

        // Center the spine object on screen.
        spineboy.x = window.innerWidth / 2;
        spineboy.y = window.innerHeight / 2;

        // Set animation "run" on track 0, looped.
        spineboy.state.setAnimation(0, "animation", true);

        // Add the display object to the stage.
        app.stage.addChild(spineboy);

        app.ticker.add((_delta) => {
            spineboy.update(_delta.deltaTime / 100);
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

<h1>Coin</h1>
