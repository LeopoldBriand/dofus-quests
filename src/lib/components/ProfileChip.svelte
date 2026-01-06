<script lang="ts">
    export let name: string;
    export let image: string;
</script>

<span class="chip">
    <img src={image} alt={name} draggable="false" />
    <span class="chip-name">{name}</span>
</span>

<style>
.chip {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    padding: 4px 8px;
    border-radius: 999px;
    background: var(--chip-bg, #2a2a2a);
    color: #fff;
    font-size: 0.85rem;
    white-space: nowrap;
}

.chip img {
    width: 18px;
    height: 18px;
    border-radius: 50%;
    object-fit: cover;
}

.chip-name {
    line-height: 1;
}
</style>
