<script lang="ts">
    import {createForm} from "felte";

    export let initialValues;
    export let onBack: undefined;
    export let onSubmit = (data) => {};

    const {form} = createForm({
        initialValues,
        onSubmit,
    });
</script>

<form use:form>
    <fieldset>
        <legend>Welcome</legend>
    </fieldset>
    <button type="submit">
        Next
    </button>
</form>