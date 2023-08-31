<script setup lang="ts">
import {useEditor, EditorContent, Editor} from "@tiptap/vue-3"
import Link from "@tiptap/extension-link"
import StarterKit from "@tiptap/starter-kit"

/**
 * StarterKit以及包含了 Document、Paragraph 和 Text
 * 这里面基本包含了全部的基本东西
 import Document from '@tiptap/extension-document'
 import ListItem from '@tiptap/extension-list-item'
 import OrderedList from '@tiptap/extension-ordered-list'
 import Paragraph from '@tiptap/extension-paragraph'
 import Text from '@tiptap/extension-text'
 */
import {ref, ShallowRef, watchEffect} from "vue"

const props = defineProps({
    modelValue: {
        required: true,
        type: String,
        default: "",
    },
    modelJson: {
        type: Object,
        default: () => {
        }
    },
    limit: {
        type: Number,
        default: 10000,
    },
})

const emit = defineEmits(["update:modelValue", "update:modelJson", "focus"])


const editor: ShallowRef<Editor | undefined> = useEditor({
    extensions: [
        StarterKit,
        Link.configure({
            autolink: false,
            openOnClick: false,
        }),
    ],
    editorProps: {
        attributes: {
            class: "ty-editor",
        }
    },
    content: props.modelValue,
    onUpdate() {
        emit("update:modelValue", editor.value?.getHTML() ?? "")
        emit("update:modelJson", editor.value?.getJSON() ?? {})
    },
    onFocus() {
        emit("focus")
        // emit("update:charNum", characters.value)
        editorIsFocus.value = true
    },
    onBlur() {
        editorIsFocus.value = false
    },
})

watchEffect(() => {
    if (props.modelValue === editor.value?.getHTML()) {
        return
    }
    editor.value?.commands.setContent(props.modelValue)
})

function setLink() {
    const previousUrl = this.editor.getAttributes('link').href
    const url = window.prompt('URL', previousUrl)

    // cancelled
    if (url === null) {
        return
    }

    // empty
    if (url === '') {
        this.editor
            .chain()
            .focus()
            .extendMarkRange('link')
            .unsetLink()
            .run()

        return
    }

    // update link
    this.editor
        .chain()
        .focus()
        .extendMarkRange('link')
        .setLink({href: url})
        .run()
}

const editorIsFocus = ref(false)

</script>

<template>

    <div v-if="editor">
        <button @click="editor.chain().focus().toggleBold().run()"
                :disabled="!editor.can().chain().focus().toggleBold().run()"
                :class="{ 'is-active': editor.isActive('bold') }">
            bold
        </button>
        <button @click="editor.chain().focus().toggleItalic().run()"
                :disabled="!editor.can().chain().focus().toggleItalic().run()"
                :class="{ 'is-active': editor.isActive('italic') }">
            italic
        </button>
        <button @click="editor.chain().focus().toggleBulletList().run()"
                :class="{ 'is-active': editor.isActive('bulletList') }">
            bullet list
        </button>
        <button @click="editor.chain().focus().toggleOrderedList().run()"
                :class="{ 'is-active': editor.isActive('orderedList') }">
            ordered list
        </button>


        <button @click="editor.chain().focus().sinkListItem('listItem').run()"
                :disabled="!editor.can().sinkListItem('listItem')">
            sink list item
        </button>
        <button type="button" @click="editor.chain().focus().liftListItem('listItem').run()"
                :disabled="!editor.can().liftListItem('listItem')">
            lift list item
        </button>

        <button type="button" @click="setLink" :class="{ 'is-active': editor.isActive('link') }">
            link
        </button>

        <button type="button" @click="editor.chain().focus().unsetAllMarks().run()">
            clear formatting
        </button>

        <button @click="editor.chain().focus().undo().run()" :disabled="!editor.can().chain().focus().undo().run()">
            undo
        </button>
        <button @click="editor.chain().focus().redo().run()" :disabled="!editor.can().chain().focus().redo().run()">
            redo
        </button>
    </div>
    <editor-content :editor="editor"/>
</template>

<style lang="scss">

</style>

