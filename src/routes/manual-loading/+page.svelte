<script lang="ts">
    import { Application, Assets } from "pixi.js";
    import {
        Spine,
        AtlasAttachmentLoader,
        SkeletonBinary,
    } from "@esotericsoftware/spine-pixi-v8";
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
        Assets.add({ alias: "spineboyData", src: "/assets/spineboy-pro.skel" });
        Assets.add({
            alias: "spineboyAtlas",
            src: "/assets/spineboy-pro.atlas",
        });
        await Assets.load(["spineboyData", "spineboyAtlas"]);

        // Manually load the data and create a Spine display object from it using
        // the Spine core API. This will not use the interal cache like Spine.from(),
        // so you have to cache data yourself.
        const atlas = Assets.get("spineboyAtlas");
        const attachmentLoader = new AtlasAttachmentLoader(atlas);
        const binaryLoader = new SkeletonBinary(attachmentLoader);
        binaryLoader.scale = 0.5;
        const skeletonData = binaryLoader.readSkeletonData(
            Assets.get("spineboyData"),
        );
        const spineboy = new Spine(skeletonData);

        // Set the default mix time to use when transitioning
        // from one animation to the next.
        spineboy.state.data.defaultMix = 0.2;

        // Center the spine object on screen.
        spineboy.x = window.innerWidth / 2;
        spineboy.y = window.innerHeight / 2 + spineboy.getBounds().height / 2;

        // Set animation "cape-follow-example" on track 0, looped.
        spineboy.state.setAnimation(0, "run", true);

        // Add the display object to the stage.
        app.stage.addChild(spineboy);
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

<h1>SpineBoy with Pixi.js & SvelteKit</h1>
