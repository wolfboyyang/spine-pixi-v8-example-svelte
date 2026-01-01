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
        Assets.add({ alias: "sackData", src: "/assets/sack-pro.skel" });
        Assets.add({ alias: "sackAtlas", src: "/assets/sack-pma.atlas" });
        await Assets.load(["sackData", "sackAtlas"]);

        // Create the spine display object
        const sack = Spine.from({
            skeleton: "sackData",
            atlas: "sackAtlas",
            scale: 0.2,
        });

        // Center the spine object on screen.
        sack.x = window.innerWidth / 2;
        sack.y = window.innerHeight / 2 + sack.getBounds().height / 2;

        // Set animation "cape-follow-example" on track 0, looped.
        sack.state.setAnimation(0, "cape-follow-example", true);

        // Add the display object to the stage.
        app.stage.addChild(sack);
    });
</script>

<h1>Physics</h1>
