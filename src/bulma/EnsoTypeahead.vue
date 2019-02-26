<template>
    <typeahead :i18n="__"
        v-bind="$attrs"
        v-on="$listeners"/>
</template>

<script>
import Typeahead from './Typeahead.vue';

export default {
    components: { Typeahead },
};
</script>
