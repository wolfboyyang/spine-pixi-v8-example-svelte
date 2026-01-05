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
        Assets.add({
            alias: "cloudPotData",
            src: "/assets/cloud-pot.skel",
        });
        Assets.add({
            alias: "cloudPotAtlas",
            src: "/assets/cloud-pot-pma.atlas",
        });
        await Assets.load(["cloudPotData", "cloudPotAtlas"]);

        // Create the spine display object
        const cloudPot = Spine.from({
            skeleton: "cloudPotData",
            atlas: "cloudPotAtlas",
            scale: 0.25,
        });

        // Center the spine object on screen.
        cloudPot.x = window.innerWidth / 2;
        cloudPot.y = window.innerHeight / 2 + cloudPot.getBounds().height / 4;

        // Set animation "playing-in-the-rain" on track 0, looped.
        cloudPot.state.setAnimation(0, "playing-in-the-rain", true);

        // Add the display object to the stage.
        app.stage.addChild(cloudPot);
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

<h1>Physics 4</h1>
