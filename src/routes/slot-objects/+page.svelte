<script lang="ts">
    import { Application, Assets, Sprite, Text, Container } from "pixi.js";
    import { Spine, type Slot } from "@esotericsoftware/spine-pixi-v8";
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
        Assets.add({
            alias: "raptor_jaw",
            src: "/assets/raptor-jaw-tooth.png",
        });

        await Assets.load(["spineboyData", "spineboyAtlas", "raptor_jaw"]);

        // Create the spine display object
        const spineboy = Spine.from({
            skeleton: "spineboyData",
            atlas: "spineboyAtlas",
            scale: 0.25,
        });

        // Set the default mix time to use when transitioning
        // from one animation to the next.
        spineboy.state.data.defaultMix = 0.2;

        // Set animation "run" on track 0, looped.
        spineboy.state.setAnimation(0, "walk", true);

        // Center the spine object on screen.
        spineboy.x = window.innerWidth / 2;
        spineboy.y = window.innerHeight / 2 + spineboy.getBounds().height / 2;

        // Add the display object to the stage.
        app.stage.addChild(spineboy);

        const tooth1 = Sprite.from("raptor_jaw");
        const tooth2 = Sprite.from("raptor_jaw");
        const text = new Text({ text: "Text GUN" });

        const toothContainer = new Container();
        toothContainer.addChild(tooth1);
        toothContainer.label = "tooth";
        text.label = "text";

        // putting logo2 on top of the hand using slot directly and remove the attachment hand
        let frontFist: Slot | null;
        setTimeout(() => {
            frontFist = spineboy.skeleton.findSlot("front-foot");
            if (frontFist) {
                tooth1.x = -10;
                tooth1.y = -70;
                spineboy.addSlotObject(frontFist, toothContainer);
                frontFist.setAttachment(null);
            }
        }, 1000);

        // scaling the bone, will scale the pixi object too
        setTimeout(() => {
            if (frontFist) {
                frontFist.bone.scaleX = 0.5;
                frontFist.bone.scaleY = 0.5;
            }
        }, 2000);

        // adding a pixi text in a slot using slot index
        let mouth;
        setTimeout(() => {
            spineboy.addSlotObject("gun", text);
        }, 4000);

        // adding one pixi object to an already "occupied" slot will remove the old one,
        // and move the given one to the slot
        setTimeout(() => {
            spineboy.addSlotObject("gun", toothContainer);
        }, 5000);

        // adding multiple DisplayObjects to a slot using a Container to control their offset, size, ...
        setTimeout(() => {
            toothContainer.addChild(tooth2);
            tooth2.x = 30;
            tooth2.y = -70;
            tooth2.angle = 30;
            tooth2.tint = 0xff5500;
        }, 6000);

        // removing the container won't automatically destroy the displayObject contained, so take care of them
        setTimeout(() => {
            spineboy.removeSlotObject("gun");
            console.log(toothContainer.destroyed);
            console.log(tooth1.destroyed);
            console.log(tooth2.destroyed);
            toothContainer.destroy();
            tooth1.destroy();
            console.log(toothContainer.destroyed);
            console.log(tooth1.destroyed);
            console.log(tooth2.destroyed);
        }, 7000);

        setTimeout(() => {
            spineboy.addSlotObject("gun", tooth2);
        }, 8000);

        // removing a specific slot object
        setTimeout(() => {
            spineboy.removeSlotObject("gun"); // tooth2
            tooth2.destroy();
        }, 9000);

        // resetting the slot with the original attachment
        setTimeout(() => {
            if (frontFist) {
                frontFist.setToSetupPose();
                frontFist.bone.setToSetupPose();
            }
        }, 10000);

        // showing an animation with clipping -> Pixi masks will be created
        // for clipping attachments having slot objects
        setTimeout(() => {
            spineboy.state.setAnimation(0, "portal", true);
            spineboy.update(0);
            const tooth3 = Sprite.from("raptor_jaw");
            tooth3.scale.set(2);
            tooth3.x = -60;
            tooth3.y = 120;
            tooth3.angle = 180;
            const foot1 = new Container();
            foot1.addChild(tooth3);
            spineboy.addSlotObject("rear-foot", foot1);
        }, 11000);
    });
</script>

<h1>Slot Objects</h1>
