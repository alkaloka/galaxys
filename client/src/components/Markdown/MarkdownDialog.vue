<script setup lang="ts">
import BootstrapVue from "bootstrap-vue";
import { storeToRefs } from "pinia";
import Vue, { computed, ref } from "vue";

import { publishedHistoriesFetcher } from "@/api/histories";
import { invocationsFetcher } from "@/api/invocations";
import { jobsFetcher } from "@/api/jobs";
import { workflowsFetcher } from "@/api/workflows";
import { useHistoryStore } from "@/stores/historyStore";

import MarkdownSelector from "./MarkdownSelector.vue";
import MarkdownVisualization from "./MarkdownVisualization.vue";
import DataDialog from "@/components/DataDialog/DataDialog.vue";
import BasicSelectionDialog from "@/components/SelectionDialog/BasicSelectionDialog.vue";
import DatasetCollectionDialog from "@/components/SelectionDialog/DatasetCollectionDialog.vue";

Vue.use(BootstrapVue);

interface MarkdownDialogProps {
    argumentName?: string;
    argumentType?: string;
    argumentPayload?: object;
    labels?: string[];
    useLabels: boolean;
}

const props = withDefaults(defineProps<MarkdownDialogProps>(), {
    argumentName: undefined,
    argumentType: undefined,
    argumentPayload: undefined,
});

const emit = defineEmits<{
    (e: "onCancel"): void;
    (e: "onInsert", value: string): void;
}>();

interface SelectTitles {
    labelTitle: string;
}

type SelectType = "job_id" | "invocation_id" | "history_dataset_id" | "history_dataset_collection_id";

const selectorConfig = {
    job_id: {
        labelTitle: "Step",
    },
    invocation_id: {
        labelTitle: "Step",
    },
    history_dataset_id: {
        labelTitle: "Output",
    },
    history_dataset_collection_id: {
        labelTitle: "Output",
    },
};

const selectedShow = ref(false);
const visualizationShow = ref(false);
const workflowShow = ref(false);
const historyShow = ref(false);
const jobShow = ref(false);
const invocationShow = ref(false);
const dataShow = ref(false);
const dataCollectionShow = ref(false);
const { currentHistoryId } = storeToRefs(useHistoryStore());

const selectedLabelTitle = computed(() => {
    const config: SelectTitles = selectorConfig[props.argumentType as SelectType] as SelectTitles;
    return (config && config.labelTitle) || "Select Label";
});

function getInvocations() {
    return invocationsFetcher({});
}

function getJobs() {
    return jobsFetcher({});
}

function getWorkflows() {
    return workflowsFetcher({});
}

function getHistories() {
    return publishedHistoriesFetcher({});
}

function onData(response: string) {
    dataShow.value = false;
    emit("onInsert", `${props.argumentName}(history_dataset_id=${response})`);
}

interface ObjectReference {
    id: string;
}

function onDataCollection(response: ObjectReference) {
    dataCollectionShow.value = false;
    emit("onInsert", `${props.argumentName}(history_dataset_collection_id=${response.id})`);
}

function onJob(response: ObjectReference) {
    jobShow.value = false;
    emit("onInsert", `${props.argumentName}(job_id=${response.id})`);
}

function onInvocation(response: ObjectReference) {
    invocationShow.value = false;
    emit("onInsert", `${props.argumentName}(invocation_id=${response.id})`);
}

function onHistory(response: ObjectReference) {
    historyShow.value = false;
    emit("onInsert", `history_link(history_id=${response.id})`);
}

function onWorkflow(response: ObjectReference) {
    workflowShow.value = false;
    emit("onInsert", `${props.argumentName}(workflow_id=${response.id})`);
}

function onVisualization(response: string) {
    visualizationShow.value = false;
    emit("onInsert", response);
}

function onOk(selectedLabel: string) {
    selectedLabel = selectedLabel || "<ENTER LABEL>";
    selectedShow.value = false;
    if (props.argumentType == "history_dataset_id") {
        if (props.useLabels) {
            emit("onInsert", `${props.argumentName}(output="${selectedLabel}")`);
        } else {
            dataShow.value = true;
        }
    } else if (props.argumentType == "history_dataset_collection_id") {
        if (props.useLabels) {
            emit("onInsert", `${props.argumentName}(output="${selectedLabel}")`);
        } else {
            dataCollectionShow.value = true;
        }
    } else if (props.argumentType == "job_id") {
        if (props.useLabels) {
            emit("onInsert", `${props.argumentName}(step="${selectedLabel}")`);
        } else {
            jobShow.value = true;
        }
    } else if (props.argumentType == "invocation_id") {
        if (props.useLabels) {
            emit("onInsert", `${props.argumentName}(step="${selectedLabel}")`);
        } else {
            invocationShow.value = true;
        }
    }
}

function onCancel() {
    dataCollectionShow.value = false;
    selectedShow.value = false;
    workflowShow.value = false;
    visualizationShow.value = false;
    jobShow.value = false;
    invocationShow.value = false;
    dataShow.value = false;
    emit("onCancel");
}

if (props.argumentType == "workflow_id") {
    workflowShow.value = true;
} else if (props.argumentType == "history_id") {
    historyShow.value = true;
} else if (props.argumentType == "history_dataset_id") {
    if (props.useLabels) {
        selectedShow.value = true;
    } else {
        dataShow.value = true;
    }
} else if (props.argumentType == "history_dataset_collection_id") {
    if (props.useLabels) {
        selectedShow.value = true;
    } else {
        dataCollectionShow.value = true;
    }
} else if (props.argumentType == "invocation_id") {
    if (props.useLabels) {
        selectedShow.value = true;
    } else {
        invocationShow.value = true;
    }
} else if (props.argumentType == "job_id") {
    if (props.useLabels) {
        selectedShow.value = true;
    } else {
        jobShow.value = true;
    }
} else if (props.argumentType == "visualization_id") {
    visualizationShow.value = true;
}
</script>

<template>
    <span>
        <MarkdownSelector
            v-if="selectedShow"
            :initial-value="argumentType"
            :argument-name="argumentName"
            :labels="labels"
            :label-title="selectedLabelTitle"
            @onOk="onOk"
            @onCancel="onCancel" />
        <MarkdownVisualization
            v-else-if="visualizationShow"
            :argument-name="argumentName"
            :argument-payload="argumentPayload"
            :labels="labels"
            :use-labels="useLabels"
            :history="currentHistoryId"
            @onOk="onVisualization"
            @onCancel="onCancel" />
        <DataDialog v-else-if="dataShow" :history="currentHistoryId" format="id" @onOk="onData" @onCancel="onCancel" />
        <DatasetCollectionDialog
            v-else-if="dataCollectionShow"
            :history="currentHistoryId"
            format="id"
            @onOk="onDataCollection"
            @onCancel="onCancel" />
        <BasicSelectionDialog
            v-else-if="jobShow"
            :get-data="getJobs"
            :is-encoded="true"
            title="Job"
            label-key="id"
            @onOk="onJob"
            @onCancel="onCancel" />
        <BasicSelectionDialog
            v-else-if="invocationShow"
            :get-data="getInvocations"
            :is-encoded="true"
            title="Invocation"
            label-key="id"
            @onOk="onInvocation"
            @onCancel="onCancel" />
        <BasicSelectionDialog
            v-else-if="workflowShow"
            :get-data="getWorkflows"
            title="Workflow"
            leaf-icon="fa fa-sitemap fa-rotate-270"
            label-key="name"
            @onOk="onWorkflow"
            @onCancel="onCancel" />
        <BasicSelectionDialog
            v-else-if="historyShow"
            :get-data="getHistories"
            title="History"
            label-key="name"
            @onOk="onHistory"
            @onCancel="onCancel" />
    </span>
</template>
