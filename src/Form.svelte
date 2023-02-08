<script lang="ts">
    import {createForm} from "felte";
    import {validator} from "@felte/validator-zod";
    import {z, ZodSchema} from "zod";

    export let initialValues;
    export let schema: ZodSchema;
    export let onBack = (data: z.output<typeof schema>) =>
        () => {
            return;
        };
    export let onSubmit = (data: z.output<typeof schema>) => {
        return;
    };
    export let legend: string;

    const {form, data} = createForm({
        initialValues,
        onSubmit,
        extend: validator({schema})
    });
</script>

<form use:form>
    <button type="button" on:click={onBack($data)}>
        Back
    </button>
    <fieldset>
        <legend>{legend}</legend>

        <slot/>
    </fieldset>
    <button type="submit">
        Next
    </button>
</form>