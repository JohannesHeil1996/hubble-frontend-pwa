<template>
    <div class="text-excerpt">
        <div :class="{ collapsed: isCollapsed && loaded }" class="text" v-html="truncatedText" />
        <div v-if="isCollapsed && loaded" class="show-more" @click="toggleCollapse()" v-text="$t('Continue reading')" />
    </div>
</template>

<script>
import _ from 'lodash';

export default {
    name: 'TextExcerpt',

    props: {
        text: {
            type: String,
            required: true,
        },
        limit: {
            type: Number,
            required: true,
        },
    },

    data() {
        return {
            isCollapsed: this.limit <= this.text.length,
            onlyTruncateWords: true,
            loaded: false,
        };
    },

    computed: {
        truncatedText: function () {
            if (this.isCollapsed) {
                return this.truncateText(this.text);
            }

            return this.text;
        },
    },

    created() {
        this.loaded = true;
    },

    methods: {
        toggleCollapse: function () {
            this.isCollapsed = !this.isCollapsed;
        },
        truncateText: function (text) {
            let truncatedText;

            truncatedText = text.substring(0, this.limit);

            // only truncate whole words
            if (this.onlyTruncateWords) {
                let lastIndex = truncatedText.lastIndexOf(' ');
                truncatedText = truncatedText.substring(0, _.min([truncatedText.length, lastIndex]));
            }

            return truncatedText + '...';
        },
    },
};
</script>
