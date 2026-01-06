<script lang="ts">
    import type { Snippet } from 'svelte'

    const {
        children,
        classes,
        grow = undefined,
    }: {
        children: Snippet
        classes?: string
        grow?: number | undefined
    } = $props()
</script>

<div class={classes} style={grow ? `flex: ${grow} 1 0%; min-width: 0;` : undefined}>
    {@render children()}
</div>

<style>
    div {
        display: flex;
        flex-direction: column;
        gap: 6px;
        height: 100%;
    }
</style>
