<script lang="ts">
    import { Application, Assets, Graphics, Text } from "pixi.js";
    import {
        Spine,
        SetupPoseBoundsProvider,
        SkinsAndAnimationBoundsProvider,
        AABBRectangleBoundsProvider,
    } from "@esotericsoftware/spine-pixi-v8";
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
        const spineboy1 = Spine.from({
            skeleton: "spineboyData",
            atlas: "spineboyAtlas",
            scale: 0.2,
        });

        const spineboy2 = Spine.from({
            skeleton: "spineboyData",
            atlas: "spineboyAtlas",
            scale: 0.2,
            boundsProvider: new SetupPoseBoundsProvider(),
        });

        const spineboy3 = Spine.from({
            skeleton: "spineboyData",
            atlas: "spineboyAtlas",
            scale: 0.2,
            boundsProvider: new SkinsAndAnimationBoundsProvider(
                "portal",
                undefined,
                undefined,
                false,
            ),
        });

        const spineboy4 = Spine.from({
            skeleton: "spineboyData",
            atlas: "spineboyAtlas",
            scale: 0.2,
            boundsProvider: new SkinsAndAnimationBoundsProvider(
                "portal",
                undefined,
                undefined,
                true,
            ),
        });

        const spineboy5 = Spine.from({
            skeleton: "spineboyData",
            atlas: "spineboyAtlas",
            scale: 0.2,
            boundsProvider: new AABBRectangleBoundsProvider(
                -100,
                -100,
                100,
                100,
            ),
        });

        const maxHeight = spineboy3.getBounds().height;
        const scaleFactor = 1 / ((maxHeight * 5) / window.innerHeight);
        const scaledMaxHeight = maxHeight * scaleFactor;

        const texts = [
            "Default bounds: dynamic, recomputed when queried",
            "Set up pose bound: fixed, based on setup pose",
            "Skin and animations based bound: fixed, the max AABB rectangle containing the skeleton with the given skin and given animations (clipping is ignored)",
            "Skin and animations based bound: same as above, but with clipping true. The bounds is smaller because clipped attachments' parts are not considered",
            "AABB Rectangle bounds: fixed, manually provided bounds. The origin is in skeleton root and size are in skeleton space",
        ];

        const pointerOn: boolean[] = [];

        const elements = [
            spineboy1,
            spineboy2,
            spineboy3,
            spineboy4,
            spineboy5,
        ].map((spineboy, i) => {
            const x = 300 * scaleFactor;

            // spineboy placement
            spineboy.scale.set(scaleFactor);
            spineboy.state.setAnimation(0, "portal", true);
            spineboy.x = x;
            spineboy.y =
                70 * scaleFactor + (window.innerHeight / 10) * (1 + 2 * i);
            app.stage.addChild(spineboy);

            // yellow rectangle to show bounds
            const graphics = new Graphics();
            app.stage.addChild(graphics);

            // text
            const basicText = new Text({
                text: texts[i],
                style: {
                    fontSize: 20 * scaleFactor,
                    fill: "white",
                    wordWrap: true,
                    wordWrapWidth: 400 * scaleFactor,
                },
            });
            basicText.x = x + scaledMaxHeight + 0 * scaleFactor;
            basicText.y = scaledMaxHeight * (i + 0.5);
            basicText.anchor.set(0, 0.5);
            app.stage.addChild(basicText);

            // pointer events
            spineboy.eventMode = "static";
            spineboy.cursor = "pointer";
            spineboy.on("pointerenter", () => (pointerOn[i] = true));
            spineboy.on("pointerleave", () => (pointerOn[i] = false));

            return { spineboy, graphics };
        });

        app.ticker.add((_delta) => {
            elements.forEach(({ spineboy, graphics }, i) => {
                const bound = spineboy.getBounds();
                graphics
                    .clear()
                    .rect(bound.x, bound.y, bound.width, bound.height)
                    .stroke({ width: 2, color: 0xfeeb77 })
                    .fill({ color: 0xff0000, alpha: pointerOn[i] ? 0.2 : 0 });
            });
        });
    });
</script>

<h1>Bounds</h1>
