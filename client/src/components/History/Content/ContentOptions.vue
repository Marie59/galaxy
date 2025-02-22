<template>
    <span class="align-self-start btn-group align-items-baseline">
        <!-- Special case for collections -->
        <b-button
            v-if="isCollection && canShowCollectionDetails"
            class="collection-job-details-btn px-1"
            title="Show Details"
            size="sm"
            variant="link"
            :href="showCollectionDetailsUrl"
            @click.prevent.stop="$emit('showCollectionInfo')">
            <icon icon="info-circle" />
        </b-button>
        <!-- Common for all content items -->
        <b-button
            v-if="isDataset"
            :disabled="displayDisabled"
            :title="displayButtonTitle"
            :tabindex="tabindex"
            class="display-btn px-1"
            size="sm"
            variant="link"
            :href="displayUrl"
            @click.prevent.stop="onDisplay($event)">
            <icon icon="eye" />
        </b-button>
        <b-button
            v-if="writable && isHistoryItem"
            :disabled="editDisabled"
            :title="editButtonTitle"
            :tabindex="tabindex"
            class="edit-btn px-1"
            size="sm"
            variant="link"
            :href="editUrl"
            @click.prevent.stop="$emit('edit')">
            <icon icon="pen" />
        </b-button>
        <b-button
            v-if="writable && isHistoryItem && !isDeleted"
            :tabindex="tabindex"
            class="delete-btn px-1"
            title="Delete"
            size="sm"
            variant="link"
            @click.stop="onDelete($event)">
            <icon v-if="isDataset" icon="trash" />
            <b-dropdown v-else ref="deleteCollectionMenu" size="sm" variant="link" no-caret toggle-class="p-0 m-0">
                <template v-slot:button-content>
                    <icon icon="trash" />
                </template>
                <b-dropdown-item title="Delete collection only" @click.prevent.stop="onDeleteItem">
                    <icon icon="file" />
                    Collection only
                </b-dropdown-item>
                <b-dropdown-item title="Delete collection and elements" @click.prevent.stop="onDeleteItemRecursively">
                    <icon icon="copy" />
                    Collection and elements
                </b-dropdown-item>
            </b-dropdown>
        </b-button>
        <b-button
            v-if="writable && isHistoryItem && isDeleted"
            :tabindex="tabindex"
            class="undelete-btn px-1"
            title="Undelete"
            size="sm"
            variant="link"
            @click.stop="$emit('undelete')">
            <icon icon="trash-restore" />
        </b-button>
        <b-button
            v-if="writable && isHistoryItem && !isVisible"
            :tabindex="tabindex"
            class="unhide-btn px-1"
            title="Unhide"
            size="sm"
            variant="link"
            @click.stop="$emit('unhide')">
            <icon icon="eye-slash" />
        </b-button>
    </span>
</template>

<script>
import { prependPath } from "@/utils/redirect";
export default {
    props: {
        writable: { type: Boolean, default: true },
        isDataset: { type: Boolean, required: true },
        isDeleted: { type: Boolean, default: false },
        isHistoryItem: { type: Boolean, required: true },
        isVisible: { type: Boolean, default: true },
        state: { type: String, default: "" },
        itemUrls: { type: Object, required: true },
        keyboardSelectable: { type: Boolean, default: true },
    },
    computed: {
        displayButtonTitle() {
            if (this.displayDisabled) {
                return "This dataset is not yet viewable.";
            }
            return "Display";
        },
        displayDisabled() {
            return ["discarded", "new", "upload", "queued"].includes(this.state);
        },
        editButtonTitle() {
            if (this.editDisabled) {
                return "This dataset is not yet editable.";
            }
            return "Edit attributes";
        },
        editDisabled() {
            return ["discarded", "new", "upload", "queued", "running", "waiting"].includes(this.state);
        },
        displayUrl() {
            return prependPath(this.itemUrls.display);
        },
        editUrl() {
            return prependPath(this.itemUrls.edit);
        },
        isCollection() {
            return !this.isDataset;
        },
        canShowCollectionDetails() {
            return !!this.itemUrls.showDetails;
        },
        showCollectionDetailsUrl() {
            return prependPath(this.itemUrls.showDetails);
        },
        tabindex() {
            return this.keyboardSelectable ? "0" : "-1";
        },
    },
    methods: {
        onDisplay($event) {
            // Wrap display handler to allow ctrl/meta click to open in new tab
            // instead of triggering display.
            if ($event.ctrlKey || $event.metaKey) {
                window.open(this.displayUrl, "_blank");
            } else {
                this.$emit("display");
            }
        },
        onDelete() {
            if (this.isCollection) {
                this.$refs.deleteCollectionMenu.show();
            } else {
                this.onDeleteItem();
            }
        },
        onDeleteItem() {
            this.$emit("delete");
        },
        onDeleteItemRecursively() {
            const recursive = true;
            this.$emit("delete", recursive);
        },
    },
};
</script>
<style lang="css">
.dropdown-menu .dropdown-item {
    font-weight: normal;
}
</style>
```
