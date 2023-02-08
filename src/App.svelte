<script lang="ts">
    import {QueryClient, QueryClientProvider} from "@tanstack/svelte-query";
    import Routes from "./Routes.svelte";

    const queryClient = new QueryClient();
</script>

<QueryClientProvider client={queryClient}>
    <Routes/>
</QueryClientProvider>