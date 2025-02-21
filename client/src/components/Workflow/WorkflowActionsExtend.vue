<script setup lang="ts">
import { library } from "@fortawesome/fontawesome-svg-core";
import { faStar as farStar } from "@fortawesome/free-regular-svg-icons";
import {
    faCaretDown,
    faCopy,
    faDownload,
    faFileExport,
    faShareAlt,
    faStar,
    faTrashRestore,
} from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { BButton } from "bootstrap-vue";
import { storeToRefs } from "pinia";
import { computed } from "vue";
import { useRouter } from "vue-router/composables";

import { copyWorkflow, undeleteWorkflow } from "@/components/Workflow/workflows.services";
import { useConfirmDialog } from "@/composables/confirmDialog";
import { Toast } from "@/composables/toast";
import { useUserStore } from "@/stores/userStore";
import { withPrefix } from "@/utils/redirect";

library.add(faCaretDown, faCopy, faDownload, faFileExport, faShareAlt, farStar, faStar, faTrashRestore);

interface Props {
    workflow: any;
    menu?: boolean;
    published?: boolean;
    buttonSize?: "sm" | "md" | "lg";
}

const props = withDefaults(defineProps<Props>(), {
    buttonSize: "sm",
});

const emit = defineEmits<{
    (e: "refreshList", a?: boolean): void;
}>();

const router = useRouter();
const userStore = useUserStore();
const { confirm } = useConfirmDialog();
const { isAnonymous } = storeToRefs(useUserStore());

const downloadUrl = computed(() => {
    return withPrefix(`/api/workflows/${props.workflow.id}/download?format=json-download`);
});
const shared = computed(() => {
    if (userStore.currentUser) {
        return userStore.currentUser.username !== props.workflow.owner;
    } else {
        return false;
    }
});

function onShare() {
    router.push(`/workflows/sharing?id=${props.workflow.id}`);
}

async function onCopy() {
    const confirmed = await confirm("Are you sure you want to make a copy of this workflow?", "Copy workflow");

    if (confirmed) {
        await copyWorkflow(props.workflow.id, props.workflow.owner);
        emit("refreshList", true);
        Toast.success("Workflow copied");
    }
}

async function onRestore() {
    const confirmed = await confirm("Are you sure you want to restore this workflow?", "Restore workflow");

    if (confirmed) {
        await undeleteWorkflow(props.workflow.id);
        emit("refreshList", true);
        Toast.info("Workflow restored");
    }
}
</script>

<template>
    <div class="workflow-actions-extend flex-gapx-1">
        <BButtonGroup>
            <BButton
                v-if="!isAnonymous && !shared && !workflow.deleted"
                id="workflow-copy-button"
                v-b-tooltip.hover.noninteractive
                :size="buttonSize"
                title="Copy"
                variant="outline-primary"
                @click="onCopy">
                <FontAwesomeIcon :icon="faCopy" fixed-width />
                <span class="compact-view">Copy</span>
            </BButton>

            <BButton
                v-if="!workflow.deleted"
                id="workflow-download-button"
                v-b-tooltip.hover.noninteractive
                :size="buttonSize"
                title="Download workflow in .ga format"
                variant="outline-primary"
                :href="downloadUrl">
                <FontAwesomeIcon :icon="faDownload" fixed-width />
                <span class="compact-view">Download</span>
            </BButton>

            <BButton
                v-if="!shared && !workflow.deleted"
                id="workflow-share-button"
                v-b-tooltip.hover.noninteractive
                :size="buttonSize"
                title="Share"
                variant="outline-primary"
                @click="onShare">
                <FontAwesomeIcon :icon="faShareAlt" fixed-width />
                <span class="compact-view">Share</span>
            </BButton>

            <BButton
                v-if="workflow.deleted"
                id="restore-button"
                v-b-tooltip.hover.noninteractive
                :size="buttonSize"
                title="Restore"
                variant="outline-primary"
                @click="onRestore">
                <FontAwesomeIcon :icon="faTrashRestore" fixed-width />
                <span class="compact-view">Restore</span>
            </BButton>
        </BButtonGroup>
    </div>
</template>

<style scoped lang="scss">
@import "breakpoints.scss";

.workflow-actions-extend {
    display: flex;
    align-items: baseline;
    flex-wrap: wrap;
    justify-content: flex-end;

    @container (max-width: #{$breakpoint-md}) {
        .compact-view {
            display: none;
        }
    }
}
</style>
