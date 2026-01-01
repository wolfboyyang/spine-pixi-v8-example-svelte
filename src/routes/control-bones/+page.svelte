<script lang="ts">
    import { Application, Assets, Graphics } from "pixi.js";
    import { type Bone, Spine } from "@esotericsoftware/spine-pixi-v8";
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

        app.stage.eventMode = "static";
        app.stage.hitArea = app.screen;
        let dragObject: Graphics | null = null;
        let lastX = -1,
            lastY = -1;
        const endDrag = () => (dragObject = null);
        app.stage
            .on("pointerup", endDrag)
            .on("pointerupoutside", endDrag)
            .on("pointermove", ({ x, y }) => {
                if (dragObject) {
                    dragObject.x += x - lastX;
                    dragObject.y += y - lastY;
                    lastX = x;
                    lastY = y;
                }
            });

        // Pre-load the skeleton data and atlas. You can also load .json skeleton data.
        Assets.add({
            alias: "stretchymanData",
            src: "/assets/stretchyman-pro.skel",
        });
        Assets.add({
            alias: "stretchymanAtlas",
            src: "/assets/stretchyman-pma.atlas",
        });
        await Assets.load(["stretchymanData", "stretchymanAtlas"]);

        // Create the spine display object
        const stretchyman = Spine.from({
            skeleton: "stretchymanData",
            atlas: "stretchymanAtlas",
            scale: 0.75,
        });

        // Set the default mix time to use when transitioning
        // from one animation to the next.
        stretchyman.state.data.defaultMix = 0.2;

        // Set animation "run" on track 0, looped.
        stretchyman.state.setAnimation(0, "idle", true);

        // Center the spine object on screen.
        stretchyman.x = window.innerWidth / 2;
        stretchyman.y =
            window.innerHeight / 2 + stretchyman.getBounds().height / 2;
        app.stage.addChild(stretchyman);

        const controlBoneNames = [
            "back-arm-ik-target",
            "back-leg-ik-target",
            "front-arm-ik-target",
            "front-leg-ik-target",
        ];
        const controlBones: { bone: Bone; control: Graphics }[] = [];

        // wait a frame as pixi bounds do not work until rendered
        await new Promise((resolve) => requestAnimationFrame(resolve));

        for (var i = 0; i < controlBoneNames.length; i++) {
            const bone = stretchyman.skeleton.findBone(controlBoneNames[i]);
            if (!bone) continue;
            const point = { x: bone.worldX, y: bone.worldY };
            stretchyman.skeletonToPixiWorldCoordinates(point);

            const control = new Graphics().circle(0, 0, 6).fill("#ff00ff");
            control.x = point.x;
            control.y = point.y;
            controlBones.push({ bone, control });
            app.stage.addChild(control);

            control.interactive = true;
            control.on("pointerdown", ({ x, y }) => {
                dragObject = control;
                lastX = x;
                lastY = y;
            });
        }

        const point = { x: 0, y: 0 };
        stretchyman.beforeUpdateWorldTransforms = () => {
            for (let { bone, control } of controlBones) {
                point.x = control.x;
                point.y = control.y;
                stretchyman.pixiWorldCoordinatesToBone(point, bone);
                bone.x = point.x;
                bone.y = point.y;
            }
        };
    });
</script>

<h1>Control bones</h1>
