<script lang="ts">
    // Can either make a generic hook or a component for less boilerplate

    import Form from "./Form.svelte";
    import {z} from "zod";

    const schema = z.object({
        photo: z.literal("base64"),
    });

    export let initialValues;
    export let onBack = (data: z.output<typeof schema>) =>
        () => {
        return;
        };
    export let onSubmit = (data: z.output<typeof schema>) => {};
    const legend = "DocumentPhoto";
</script>

<Form
        {initialValues}
        {schema}
        {legend}
        {onSubmit}
        {onBack}
>
    <input type="hidden" name="photo" value="base64"/>
</Form>