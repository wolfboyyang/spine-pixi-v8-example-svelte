<script lang="ts">
    import { goto } from "$app/navigation";
    import { page } from "$app/state";

    const routes = [
        { path: "/", label: "Home" },
        { path: "/bounds", label: "Bounds" },
        { path: "/bunnymark", label: "Bunnymark" },
        { path: "coin/", label: "Coin" },
        { path: "control-bones/", label: "Control Bones" },
        { path: "dragon/", label: "Dragon" },
        { path: "events/", label: "Events" },
        { path: "inline-loading/", label: "Inline Loading" },
        { path: "manual-loading/", label: "Manual Loading" },
        { path: "mouse-following/", label: "Mouse Following" },
        { path: "mix-and-match/", label: "Mix and Match" },
        { path: "physics/", label: "Physics" },
        { path: "physics2/", label: "Physics 2" },
        { path: "physics3/", label: "Physics 3" },
        { path: "physics4/", label: "Physics 4" },
        { path: "simple-input/", label: "Simple Input" },
        { path: "slot-objects/", label: "Slot Objects" },
        {
            path: "slot-objects-scale-rotation/",
            label: "Slot Objects Scale Rotation",
        },
    ];
</script>

<header class="flex justify-between">
    <div class="flex items-center">
        <ul>
            {#each routes as route}
                <li
                    aria-current={page.url.pathname === route.path
                        ? "page"
                        : undefined}
                >
                    <button onclick={() => goto(route.path)}
                        >{route.label}</button
                    >
                </li>
            {/each}
        </ul>
    </div>
</header>

<style>
    ul {
        position: relative;
        padding: 0;
        margin: 0;
        height: 3em;
        display: flex;
        justify-content: center;
        align-items: center;
        list-style: none;
        background: var(--background);
        background-size: contain;
    }

    li {
        position: relative;
        height: 100%;
    }

    li[aria-current="page"]::before {
        --size: 6px;
        content: "";
        width: 0;
        height: 0;
        position: absolute;
        top: 0;
        left: calc(50% - var(--size));
        border: var(--size) solid transparent;
        border-top: var(--size) solid var(--color-theme-1);
    }

    button {
        display: flex;
        height: 100%;
        align-items: center;
        padding: 0 0.5rem;
        color: var(--color-text);
        font-weight: 700;
        font-size: 0.8rem;
        text-transform: uppercase;
        letter-spacing: 0.1em;
        text-decoration: none;
        transition: color 0.2s linear;
    }

    button:hover {
        color: var(--color-theme-1);
    }
</style>
