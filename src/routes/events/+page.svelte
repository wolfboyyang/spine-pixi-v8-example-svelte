<script lang="ts">
    import { Application, Assets } from "pixi.js";
    import { Spine } from "@esotericsoftware/spine-pixi-v8";
    import { onMount } from "svelte";

    let message: string[] = $state([]);
    const app = new Application();

    function log(text: string) {
        message.push(text);
        console.log(text);
    }

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
        const spineboy = Spine.from({
            atlas: "spineboyAtlas",
            skeleton: "spineboyData",
            scale: 0.5,
        });

        // Set the default mix time to use when transitioning
        // from one animation to the next.
        spineboy.state.data.defaultMix = 0.2;

        // Center the spine object on screen.
        spineboy.x = window.innerWidth / 2;
        spineboy.y = window.innerHeight / 2 + spineboy.getBounds().height / 2;

        // Set animation "run" on track 0, looped.
        spineboy.state.setAnimation(0, "run", true);

        // Set callbacks to receive animation state events.
        spineboy.state.addListener({
            start: (entry) => log(`Started animation ${entry.animation?.name}`),
            interrupt: (entry) =>
                log(`Interrupted animation ${entry.animation?.name}`),
            end: (entry) => log(`Ended animation ${entry.animation?.name}`),
            dispose: (entry) =>
                log(`Disposed animation ${entry.animation?.name}`),
            complete: (entry) =>
                log(`Completed animation ${entry.animation?.name}`),
        });

        // Add a custom event listener along with an
        // unlooped animation to see the custom event logged.
        const trackEntry = spineboy.state.addAnimation(0, "walk", false, 3);
        trackEntry.listener = {
            event: (entry, event) =>
                log(
                    `Custom event for ${entry.animation?.name}: ${event.data.name}`,
                ),
        };

        spineboy.state.addAnimation(0, "run", true, 0);

        // Add the display object to the stage.
        app.stage.addChild(spineboy);
    });
</script>

<h1>events</h1>
<!-- Creates a transparent logging overlay. -->
<div id="log" class="max-h-75 select-none overflow-auto scroll-auto">
    {#each message as text}
        {text}<br />
    {/each}
</div>
