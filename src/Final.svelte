<script lang="ts">
    import {createForm} from "felte";

    export let initialValues;
    export let onBack = (data) =>
        () => {
        return;
        };
    export let onSubmit = (data) => {};

    const {form, data} = createForm({
        initialValues,
        onSubmit,
    });
</script>

<form use:form>
    <button type="button" on:click={onBack($data)}>
        Back
    </button>
    <fieldset>
        <legend>Final</legend>
    </fieldset>
    <button type="submit">
        Submit
    </button>
</form>