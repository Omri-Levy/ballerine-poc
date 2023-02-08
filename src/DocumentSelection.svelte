<script lang="ts">
    import {createForm} from "felte";
    import {z} from "zod";
    import {validator} from "@felte/validator-zod";
    import {reporter, ValidationMessage} from "@felte/reporter-svelte";

    const schema = z.object({
        documentType: z.union([
            z.literal('passport'),
            z.literal('idCard'),
            z.literal('driverLicense'),
        ]),
    });

    export let initialValues;
    export let onBack = (data: z.output<typeof schema>) => () => {
        return;
    };
    export let onSubmit = (data: z.output<typeof schema>) => {
    };
    const {form, data, createSubmitHandler} = createForm({
        initialValues,
        extend: [reporter, validator({schema})]
    });

    const onChange = createSubmitHandler({
        onSubmit,
    });
    const onClick = (event) => {
        if (event.target.type !== 'radio' || event.target.value !== $data.documentType) return;

        return createSubmitHandler({
            onSubmit,
        })(event);
    }

</script>

<form use:form>
    <button type="button" on:click={onBack($data)}>
        Back
    </button>
    <fieldset on:click={onClick} on:change={onChange}>
        <legend>DocumentSelection</legend>

        <label for="passport">
            Passport
        </label>
        <input type="radio" id="passport" name="documentType" value="passport" bind:group={$data.documentType}
        />
        <label for="id-card">
            ID Card
        </label>
        <input type="radio" id="id-card" name="documentType" value="idCard"
               bind:group={$data.documentType}
        />
        <label for="driver-license">
            Driver License
        </label>
        <input type="radio" id="driver-license" name="documentType" value="driverLicense"
               bind:group={$data.documentType}
        />
        <label for="bad-value">
            Bad Value
        </label>
        <input type="radio" id="bad-value" name="documentType" value="badValue"
               bind:group={$data.documentType}
        />
    </fieldset>
    <ValidationMessage for="documentType" let:messages={message}>
        <div style="color: red; font-weight: bold;">{message || ''}</div>
    </ValidationMessage>
</form>