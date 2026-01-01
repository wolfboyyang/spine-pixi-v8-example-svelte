<script lang="ts">
    import RangeSlider from "svelte-range-slider-pips";
    import { Application, Assets, Container, Sprite, Text } from "pixi.js";
    import {
        Bone,
        Spine,
        TextureAtlas,
        SpineTexture,
    } from "@esotericsoftware/spine-pixi-v8";
    import { onMount } from "svelte";

    const jsonSkel = {
        bones: [
            { name: "root" },
            {
                name: "pivot",
                parent: "root",
                scaleX: 1,
                scaleY: 1,
                rotation: 0,
                x: 20,
                y: -20,
            },
            {
                name: "pivot2",
                parent: "pivot",
                scaleX: 1,
                scaleY: 1,
                rotation: 0,
                x: 20,
                y: -20,
            },
            {
                name: "replaceMe",
                parent: "pivot2",
                scaleX: 1,
                scaleY: 1,
                rotation: 0,
                x: 20,
                y: -20,
            },
        ],
        slots: [{ name: "replaceMe", bone: "replaceMe", attachment: "image" }],
        skins: [
            {
                name: "default",
                attachments: {
                    replaceMe: { image: { width: 100, height: 100 } },
                },
            },
        ],
        animations: { animation: {} },
    };

    // notice there shoud be no space before "image"
    const txtAtlas = `skeleton.png
size:100,100
filter:Linear,Linear
image
bounds:0,0,100,100`;

    const slotPng =
        "data:image/png;base64,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";

    const app = new Application();
    let parentBone: Bone | null = $state(null);
    let parent2Bone: Bone | null = $state(null);
    let bone: Bone | null = $state(null);

    let parentScaleX = $state(1);
    let parentScaleY = $state(1);
    let parentRotation = $state(0);
    let parent2ScaleX = $state(1);
    let parent2ScaleY = $state(1);
    let parent2Rotation = $state(0);
    let scaleX = $state(1);
    let scaleY = $state(1);
    let rotation = $state(0);
    $effect(() => {
        if (parentBone) {
            parentBone.scaleX = parentScaleX;
            parentBone.scaleY = parentScaleY;
            parentBone.rotation = parentRotation;
        }
    });

    $effect(() => {
        if (parent2Bone) {
            parent2Bone.scaleX = parent2ScaleX;
            parent2Bone.scaleY = parent2ScaleY;
            parent2Bone.rotation = rotation;
        }
    });
    $effect(() => {
        if (bone) {
            bone.scaleX = scaleX;
            bone.scaleY = scaleY;
            bone.rotation = parent2Rotation;
        }
    });

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

        // adding skeleton to cache
        Assets.cache.set("jsonSkel", jsonSkel);

        // adding texture atlas to cache and loading the respective texture page
        const textureAtlas = new TextureAtlas(txtAtlas);
        console.log(textureAtlas);
        const texturePng = await Assets.load(slotPng);
        textureAtlas.pages[0].setTexture(SpineTexture.from(texturePng.source));
        Assets.cache.set("spineboyAtlas", textureAtlas);

        // creating spine game object
        const spineGO = Spine.from({
            skeleton: "jsonSkel",
            atlas: "spineboyAtlas",
        });
        spineGO.position.set(300, 300);
        app.stage.addChild(spineGO);

        // creating slot object
        const container = new Container();
        const sprite = Sprite.from(texturePng);
        sprite.anchor.x = 0.5;
        sprite.anchor.y = 0.5;
        sprite.tint = 0x00ff00;
        container.addChild(sprite);
        spineGO.addSlotObject("replaceMe", container);

        // adding controls
        parentBone = spineGO.skeleton.findBone("pivot");
        parent2Bone = spineGO.skeleton.findBone("pivot2");
        bone = spineGO.skeleton.findBone("replaceMe");
    });
</script>

<h1>
    This example shows that slot objects follow scale and rotation from
    ancestors too, but if a rotation on a bone occurs scale won't work anymore
    on ancestors.
</h1>
<div class="w-full inline-flex">
    parentScaleX:
    <RangeSlider
        class="w-1/2"
        float
        min={-3}
        max={3}
        step={0.1}
        bind:value={parentScaleX}
    />
</div>
<div class="w-full inline-flex">
    <p>parentScaleY</p>
    <RangeSlider
        class="w-1/2"
        float
        min={-3}
        max={3}
        step={0.1}
        bind:value={parentScaleY}
    />
</div>
<div class="w-full inline-flex">
    <p>parentRotation</p>
    <RangeSlider
        class="w-1/2"
        min={-180}
        max={180}
        step={1}
        bind:value={parentRotation}
    />
</div>
<div class="w-full inline-flex">
    <p>parent2ScaleX</p>
    <RangeSlider
        class="w-1/2"
        float
        min={-3}
        max={3}
        step={0.1}
        bind:value={parent2ScaleX}
    />
</div>
<div class="w-full inline-flex">
    <p>parent2ScaleY</p>
    <RangeSlider
        class="w-1/2"
        float
        min={-3}
        max={3}
        step={0.1}
        bind:value={parent2ScaleY}
    />
</div>
<div class="w-full inline-flex">
    <p>parent2Rotation</p>
    <RangeSlider
        class="w-1/2"
        min={-180}
        max={180}
        step={1}
        bind:value={parent2Rotation}
    />
</div>
<div class="w-full inline-flex">
    <p>scaleX</p>
    <RangeSlider
        class="w-1/2"
        float
        min={-3}
        max={3}
        step={0.1}
        bind:value={scaleX}
    />
</div>
<div class="w-full inline-flex">
    <p>scaleY</p>
    <RangeSlider
        class="w-1/2"
        float
        min={-3}
        max={3}
        step={0.1}
        bind:value={scaleY}
    />
</div>
<div class="w-full inline-flex">
    <p>rotation</p>
    <RangeSlider
        class="w-1/2"
        min={-180}
        max={180}
        step={1}
        bind:value={rotation}
    />
</div>
