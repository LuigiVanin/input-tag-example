<template>
    <div class="tag-input">
        <ul class="tag-input__tags">
            <li
                v-for="(tag, index) in tags"
                :key="tag"
                class="tag-input__tag"
                @click="setEditability(index)"
            >
                <span v-if="editingTagIndex !== index">
                    <!-- {{ index }} -->
                    {{ tag }}
                </span>
                <input
                    v-else
                    type="text"
                    class="tag-input__input"
                    placeholder="enter to add tag"
                    contenteditable
                    v-model="editingTagText"
                    @keydown.enter="saveEditedTag()"
                    @keyup.escape="editingTagIndex = -1"
                    @mouseover="($event.target as any)?.focus()"
                    @blur="resetEditability()"
                />

                <span
                    class="close"
                    @click="
                        deleteTag(index);
                        $event.stopPropagation();
                    "
                >
                    <img :src="CloseIcon" alt="" />
                </span>
            </li>
            <input
                type="text"
                class="tag-input__input"
                placeholder="Pressione ENTER para inserir"
                v-model="newtag"
                @keydown.enter="addTag"
                @focus="resetEditability()"
            />
        </ul>
        <div class="tag-input__word-count">
            {{ countTagsWords() }}/{{ props.maxWords }}
        </div>

        <span class="progress-bar">
            <span
                :class="['progress', getProgressStatus()]"
                ref="progress"
            ></span>
        </span>
    </div>
</template>
<script lang="ts" setup>
import { ref, defineEmits, onMounted } from "vue";
import CloseIcon from "../assets/close-icon.svg?url";

const tags = ref<string[]>([]);
const newtag = ref("");
const editingTagText = ref("");
const editingTagIndex = ref(-1);
const progress = ref<HTMLElement | null>(null);

const progressStatus = {
    good: 70,
    ok: 40,
    bad: 20,
};

// create function that return "bad", "ok" or "good" based on the percentage of the progress bar using progressStatus as base
const getProgressStatus = () => {
    const percentage = getPercentage();
    if (percentage >= progressStatus.good) {
        return "good";
    } else if (percentage >= progressStatus.ok) {
        return "ok";
    } else {
        return "bad";
    }
};

const emit = defineEmits(["update:tags"]);

interface Props {
    maxWords: number;
}

const props = withDefaults(defineProps<Props>(), {
    maxWords: 150,
});

onMounted(() => {
    console.log(progress.value);
});

// function that get the percentage of the progress bar
const getPercentage = () => {
    const count = countTagsWords();
    const percentage = (count / props.maxWords) * 100;
    return percentage;
};

// function that set the width of the progress bar
const setProgress = () => {
    const percentage = getPercentage();
    if (progress.value) {
        progress.value.style.width = `${percentage}%`;
    }
};

const countTagsWords = () => {
    let count = 0;
    tags.value.forEach((tag) => {
        count += tag.split("").length;
    });
    return count;
};

const emitSaveTag = () => {
    setProgress();
    emit("update:tags", tags.value);
};

const resetEditability = () => {
    editingTagIndex.value = -1;
    editingTagText.value = "";
    emitSaveTag();
};

const addTag = () => {
    if (newtag.value.trim()) {
        tags.value.push(newtag.value);
        newtag.value = "";
        emitSaveTag();
    }
};

const setEditability = (index: number) => {
    editingTagIndex.value = index;
    editingTagText.value = tags.value[index];
};

const saveEditedTag = () => {
    tags.value[editingTagIndex.value] = editingTagText.value;
    resetEditability();
};

const deleteTag = (index: number) => {
    resetEditability();
    tags.value.splice(index, 1);
    emitSaveTag();
};
</script>

<style scoped>
ul > li > input {
    background: transparent;
    border: none;
    word-wrap: break-word;
}

.tag-input {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    position: relative;
    gap: 5px;
    width: 500px;
    padding: 0.5rem 1rem;
    border: 1px solid #ccc;
    border-radius: 5px;
    background-color: #fff;
    color: #333;
    font-size: 1rem;
    font-weight: 400;
    transition: all 0.2s ease-in-out;
    padding-block: 25px;
    padding-bottom: 30px;
    overflow: hidden;
}

.progress-bar {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    height: 4px;
}

.progress-bar .progress {
    position: absolute;
    bottom: 0;
    left: 0;
    height: 4px;
    width: 0%;
    display: inline-block;
    background: rgb(65, 22, 127);
}

.progress-bar .progress.bad {
    background: #ff3838;
}

.progress-bar .progress.ok {
    background: #ffbb38;
}

.progress-bar .progress.good {
    background: #1bda67;
}

.tag-input ul > input {
    resize: none;
    min-width: 100px;
    border: none;
    outline: none;
    font-size: 0.9rem;
    flex: 1;
    width: 100%;
}

.tag-input__word-count {
    background: #f5f5f5;
    font-size: 12px;
    font-weight: bold;
    height: 30px;
    padding-inline: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 15px;
}

ul {
    /* reset list style */
    list-style: none;
    display: flex;

    /* reset spacings */
    padding: 0;
    margin: 0;
    gap: 3px;
    max-width: 100%;
    flex-wrap: wrap;
}

ul li {
    background: #e4deef;
    font-size: 0.9rem;
    line-height: 1rem;
    min-height: 2rem;
    padding-block: 2px;
    padding-inline: 1rem;
    width: auto;
    gap: 4px;
    color: #41167f;

    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 1rem;

    word-break: break-all;
    margin-bottom: 10px;
}

ul li:last-child {
}

ul span.close {
    cursor: pointer;
    color: #41167f;
    display: flex;
    align-items: center;
    justify-content: center;
}

img {
    height: 10px;
}
</style>
