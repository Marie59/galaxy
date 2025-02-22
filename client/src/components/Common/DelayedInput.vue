<template>
    <b-input-group>
        <b-input
            ref="toolInput"
            v-model="queryInput"
            class="search-query"
            size="sm"
            autocomplete="off"
            :placeholder="placeholder"
            @input="delayQuery"
            @change="setQuery"
            @keydown.esc="setQuery('')" />
        <b-input-group-append>
            <b-button
                v-if="enableAdvanced"
                v-b-tooltip.hover.bottom.noninteractive
                aria-haspopup="true"
                size="sm"
                :pressed="showAdvanced"
                :variant="showAdvanced ? 'info' : 'secondary'"
                :title="titleAdvanced | l"
                data-description="toggle advanced search"
                @click="onToggle">
                <icon v-if="showAdvanced" fixed-width icon="angle-double-up" />
                <icon v-else fixed-width icon="angle-double-down" />
            </b-button>
            <b-button
                v-b-tooltip.hover.bottom.noninteractive
                aria-haspopup="true"
                class="search-clear"
                size="sm"
                :title="titleClear | l"
                data-description="reset query"
                @click="clearBox">
                <icon v-if="loading" fixed-width icon="spinner" spin />
                <icon v-else fixed-width icon="times" />
            </b-button>
        </b-input-group-append>
    </b-input-group>
</template>
<script>
export default {
    props: {
        query: {
            type: String,
            default: "",
        },
        loading: {
            type: Boolean,
            default: false,
        },
        placeholder: {
            type: String,
            default: "Enter your search term here.",
        },
        delay: {
            type: Number,
            default: 1000,
        },
        enableAdvanced: {
            type: Boolean,
            default: false,
        },
        showAdvanced: {
            type: Boolean,
            default: false,
        },
    },
    data() {
        return {
            queryInput: null,
            queryTimer: null,
            queryCurrent: null,
            titleClear: "Clear Search (esc)",
            titleAdvanced: "Toggle Advanced Search",
        };
    },
    watch: {
        query(queryNew) {
            this.setQuery(queryNew);
        },
    },
    methods: {
        clearTimer() {
            if (this.queryTimer) {
                clearTimeout(this.queryTimer);
            }
        },
        delayQuery(query) {
            this.clearTimer();
            if (query) {
                this.queryTimer = setTimeout(() => {
                    this.setQuery(query);
                }, this.delay);
            } else {
                this.setQuery(query);
            }
        },
        setQuery(queryNew) {
            this.clearTimer();
            if (this.queryCurrent !== this.queryInput || this.queryCurrent !== queryNew) {
                this.queryCurrent = this.queryInput = queryNew;
                this.$emit("change", this.queryCurrent);
            }
        },
        clearBox() {
            this.setQuery("");
            this.$refs.toolInput.focus();
        },
        onToggle() {
            this.$emit("onToggle", !this.showAdvanced);
        },
    },
};
</script>
