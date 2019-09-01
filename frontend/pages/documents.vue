<template>
    <div class="document-page">
        <Document class="document"/>
    </div>
</template>

<script>
import Document from '@/components/renderers/Document.vue'
export default {
    components: {
        Document,
    },
}
</script>

<style lang="postcss" scoped>
.document-page {
    @apply bg-maroon-dark text-gray-200 py-8;
}
.document {
    width: 900px;
    height: 100%;
    @apply m-auto;
}
</style>