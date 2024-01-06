<script>
    export let sectionName;
    export let steps;
    export let addStep;
    export let deleteStep;
</script>

<div id="{sectionName}-section" style="display: none;" class="infomation">
    <h2>{sectionName}:</h2>
    {#if steps && steps.length > 0}
        <ul>
            {#each steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep(sectionName, step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value={sectionName} />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    {:else}
        <p>No steps added yet.</p>
    {/if}
</div>