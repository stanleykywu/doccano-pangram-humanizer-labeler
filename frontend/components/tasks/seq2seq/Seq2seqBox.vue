<template>
  <v-card>
    <v-data-table
      :headers="headers"
      :items="annotations"
      item-key="id"
      hide-default-header
      hide-default-footer
      disable-pagination
      class="elevation-1"
      @input="update"
    >
      <template #top>
        <v-text-field
          v-model="newText"
          :prepend-inner-icon="mdiPencil"
          :label="$t('annotation.newText')"
          autofocus
          single-line
          hide-details
          filled
          @keyup.enter="create"
          @compositionstart="compositionStart"
          @compositionend="compositionEnd"
        />
      </template>
      <template #[`item.text`]="{ item }">
        <div class="d-flex align-center justify-space-between">
          <v-edit-dialog>
            <span class="title" style="font-weight: 400">
              {{ item.text }}
            </span>
            <template #input>
              <v-textarea
                :value="item.text"
                :label="$t('generic.edit')"
                autofocus
                @change="update(item.id, $event)"
              />
            </template>
          </v-edit-dialog>
          <v-icon small class="ml-2" @click="copyToClipboard(item.text)">
            {{ mdiContentCopy }}
          </v-icon>
        </div>
      </template>
      <template #[`item.action`]="{ item }">
        <v-icon small @click="remove(item.id)">
          {{ mdiDeleteOutline }}
        </v-icon>
      </template>
    </v-data-table>
  </v-card>
</template>

<script lang="ts">
import Vue from 'vue'
import { mdiPencil, mdiDeleteOutline, mdiContentCopy } from '@mdi/js'

export default Vue.extend({
  props: {
    annotations: {
      type: Array,
      default: () => [],
      required: true
    }
  },

  data() {
    return {
      newText: '',
      headers: [
        {
          text: 'Text',
          align: 'left',
          value: 'text'
        },
        {
          text: 'Actions',
          align: 'right',
          value: 'action'
        }
      ],
      isComposing: false,
      hasCompositionJustEnded: false,
      mdiPencil,
      mdiDeleteOutline,
      mdiContentCopy
    }
  },

  methods: {
    update(annotationId: number, text: string) {
      if (text.length > 0) {
        this.$emit('update:annotation', annotationId, text)
      } else {
        this.remove(annotationId)
      }
    },
    create() {
      if (this.isComposing || this.hasCompositionJustEnded) {
        this.hasCompositionJustEnded = false
        return
      }
      if (this.newText.length > 0) {
        this.$emit('create:annotation', this.newText)
        this.newText = ''
      }
    },
    remove(annotationId: number) {
      this.$emit('delete:annotation', annotationId)
    },
    copyToClipboard(text: string) {
      if (navigator.clipboard && window.isSecureContext) {
        navigator.clipboard.writeText(text).then(() => {
          console.log('Text copied to clipboard')
        }).catch(() => {
          this.fallbackCopy(text)
        })
      } else {
        this.fallbackCopy(text)
      }
    },
    fallbackCopy(text: string) {
      try {
        const textarea = document.createElement('textarea')
        textarea.value = text
        textarea.style.position = 'fixed'
        textarea.style.opacity = '0'
        document.body.appendChild(textarea)
        textarea.select()
        document.execCommand('copy')
        document.body.removeChild(textarea)
        console.log('Text copied to clipboard (fallback method)')
      } catch (err) {
        console.error('Copy failed:', err)
      }
    },
    compositionStart() {
      this.isComposing = true
    },
    compositionEnd() {
      this.isComposing = false
      this.hasCompositionJustEnded = true
    }
  }
})
</script>
