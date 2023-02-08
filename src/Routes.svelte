<script lang="ts">
    import {createMachine, type StateValueMap} from "xstate";
    import {SvelteComponent} from "svelte";
    import {useMachine} from "@xstate/svelte";
    import Welcome from "./Welcome.svelte";
    import DocumentSelection from "./DocumentSelection.svelte";
    import DocumentPhoto from "./DocumentPhoto.svelte";
    import {createForm} from "felte";
    import Final from "./Final.svelte";
    import {createMutation} from "@tanstack/svelte-query";
    import Error from "./Error.svelte";
    import Success from "./Success.svelte";

    const Step = {
        WELCOME: "WELCOME",
        DOCUMENT_SELECTION: "DOCUMENT_SELECTION",
        DOCUMENT_PHOTO: "DOCUMENT_PHOTO",
        FINAL: "FINAL",
        ERROR: "ERROR",
        SUCCESS: "SUCCESS",
    } as const;

    const Action = {
        NEXT: "NEXT",
        BACK: "BACK",
        SUBMIT: "SUBMIT",
        ERROR: "ERROR",
        SUCCESS: "SUCCESS",
    } as const;

    const verify = async (data) => {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                Math.random() > 0.5 ? resolve(data) : reject("Error!")
            }, 2000);
        });
    };
    const mutation = createMutation({
        mutationFn: verify,
        onError(data) {
            send({
                type: Action.ERROR,
            });
        },
        onSuccess(data) {
            send({
                type: Action.SUCCESS,
            });
        },
    });
    const machine = createMachine({
            id: 'form',
            initial: Step.WELCOME,
            predictableActionArguments: true,
            schema: {
                context: {} as {
                    data: Record<Exclude<keyof typeof Step, 'ERROR' | 'SUCCESS'>, Record<PropertyKey, any>>;
                    pages: Record<keyof typeof Step, SvelteComponent>;
                },
                events: {} as
                    | {
                    type: typeof Action.NEXT;
                    data: Record<PropertyKey, any>;
                    state: string | StateValueMap;
                }
                    | {
                    type: typeof Action.BACK;
                    data: Record<PropertyKey, any>;
                    state: string | StateValueMap;
                }
                    | {
                    type: typeof Action.SUBMIT;
                    data: Record<PropertyKey, any>;
                    state: string | StateValueMap;
                } | {
                    type: typeof Action.ERROR;
                } | {
                    type: typeof Action.SUCCESS;
                },
            },
            context: {
                // data and pages could potentially be merged
                // there's also xstate's 'meta' which could be
                // used for data or the page component
                data: {
                    [Step.WELCOME]: {},
                    [Step.DOCUMENT_SELECTION]: {
                        documentType: undefined,
                    },
                    [Step.DOCUMENT_PHOTO]: {
                        base64: undefined,
                    },
                    [Step.FINAL]: {},
                },
                pages: {
                    [Step.WELCOME]: Welcome,
                    [Step.DOCUMENT_SELECTION]: DocumentSelection,
                    [Step.DOCUMENT_PHOTO]: DocumentPhoto,
                    [Step.FINAL]: Final,
                    [Step.ERROR]: Error,
                    [Step.SUCCESS]: Success,
                },
            },

            on: {
                [Action.ERROR]: Step.ERROR,
                [Action.SUCCESS]: Step.SUCCESS,
            },

            states: {
                [Step.WELCOME]: {
                    on: {
                        [Action.NEXT]: {
                            target: Step.DOCUMENT_SELECTION,
                            // Some actions can be set on a machine's top level 'on' property.
                            actions: Action.NEXT,
                        },
                    },
                },
                [Step.DOCUMENT_SELECTION]: {
                    on: {
                        [Action.NEXT]: {
                            target: Step.DOCUMENT_PHOTO,
                            actions: Action.NEXT,
                        },
                        [Action.BACK]: {
                            target: Step.WELCOME,
                            actions: Action.BACK,
                        },
                    },
                },
                [Step.DOCUMENT_PHOTO]: {
                    on: {
                        [Action.NEXT]: {
                            target: Step.FINAL,
                            actions: Action.NEXT,
                        },
                        [Action.BACK]: {
                            target: Step.DOCUMENT_SELECTION,
                            actions: Action.BACK,
                        },
                    },
                },
                [Step.FINAL]: {
                    on: {
                        [Action.BACK]: {
                            target: Step.DOCUMENT_PHOTO,
                            actions: {
                                type: Action.BACK,
                                target: Step.DOCUMENT_PHOTO,
                            },
                        },
                        [Action.SUBMIT]: {
                            actions: Action.SUBMIT,
                        },
                    },
                },
                [Step.ERROR]: {
                    on: {
                        [Action.BACK]: {
                            target: Step.FINAL,
                            actions: {
                                type: Action.BACK,
                                target: Step.FINAL,
                            },
                        },
                    },
                },
                [Step.SUCCESS]: {
                    on: {
                        [Action.BACK]: {
                            target: Step.WELCOME,
                            actions: {
                                type: Action.BACK,
                                target: Step.WELCOME,
                            },
                        },
                    },
                },
            }
        },
        {
            actions: {
                [Action.NEXT]: (context, event) => {
                    context.data[event.state] = event.data;
                },
                [Action.BACK]: (context, event) => {
                    context.data[event.state] = event.data;
                },
                [Action.SUBMIT]: (context) => {
                    $mutation.mutate(context.data);
                },
            },
        });
    const {send, state} = useMachine(machine);
    // Each form can also specify an onError or onSuccess
    // or svelte-query's states can be used - either can be used to fire an xstate event.
    const onSubmit = (data) => {

        // Can be handled directly in the machine's NEXT action.
        if ($state.value === Step.FINAL) {
            send({
                type: Action.SUBMIT,
                data,
                // A workaround for not having access to the state's value
                // from an action. This is a non-issue on array + index based navigation.
                state: $state.value,
            })
        }

        send({
            type: Action.NEXT,
            data,
            state: $state.value,
        })
    };
    const onBack = (data) => () => {
        send({
            type: Action.BACK,
            data,
            state: $state.value,
        })
    };
</script>

<svelte:component
        this={$state.context.pages[$state.value]}
        initialValues={$state.context.data[$state.value]}
        {onBack}
        {onSubmit}
/>

{#if $mutation.isLoading}
    Loading...
{/if}

{#if $mutation.isError}
    Error!
{/if}

{#if $mutation.isSuccess}
    Success!
{/if}