<script lang="ts">
    import { Application, Assets } from "pixi.js";
    import { Spine, Skin } from "@esotericsoftware/spine-pixi-v8";
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
            alias: "mixAndMatchData",
            src: "/assets/mix-and-match-pro.skel",
        });
        Assets.add({
            alias: "mixAndMatchAtlas",
            src: "/assets/mix-and-match-pma.atlas",
        });
        await Assets.load(["mixAndMatchData", "mixAndMatchAtlas"]);

        // Create the Spine display object
        const mixAndMatch = Spine.from({
            skeleton: "mixAndMatchData",
            atlas: "mixAndMatchAtlas",
            scale: 0.5,
        });

        // Set the default mix time to use when transitioning
        // from one animation to the next.
        mixAndMatch.state.data.defaultMix = 0.2;

        // Add animations.
        mixAndMatch.state.setAnimation(0, "walk", true);
        mixAndMatch.state.addAnimation(0, "dance", true, 1.0);
        mixAndMatch.state.addAnimation(0, "walk", true, 1.0);

        // Add a custom skin
        const skeletonData = mixAndMatch.skeleton.data;
        const skin = new Skin("custom");
        [
            skeletonData.findSkin("custom"),
            skeletonData.findSkin("nose/short"),
            skeletonData.findSkin("skin-base"),
            skeletonData.findSkin("eyes/violet"),
            skeletonData.findSkin("hair/brown"),
            skeletonData.findSkin("clothes/hoodie-orange"),
            skeletonData.findSkin("legs/pants-jeans"),
            skeletonData.findSkin("accessories/bag"),
            skeletonData.findSkin("accessories/hat-red-yellow"),
            skeletonData.findSkin("eyelids/girly"),
        ].forEach((skinData) => {
            if (skinData) skin.addSkin(skinData);
        });
        mixAndMatch.skeleton.setSkin(skin);
        mixAndMatch.skeleton.setSlotsToSetupPose();

        // Center the spine object on screen.
        mixAndMatch.x = window.innerWidth / 2;
        mixAndMatch.y =
            window.innerHeight / 2 + mixAndMatch.getBounds().height / 2;

        // Add the display object to the stage.
        app.stage.addChild(mixAndMatch);
    });
</script>

<h1>Mix and Match</h1>
