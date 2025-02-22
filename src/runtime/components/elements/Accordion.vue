<template>
  <div :class="ui.wrapper">
    <HDisclosure v-for="(item, index) in items" v-slot="{ open, close }" :key="index" :default-open="defaultOpen || item.defaultOpen">
      <HDisclosureButton :ref="() => buttonRefs[index] = close" as="template" :disabled="item.disabled">
        <slot :item="item" :index="index" :open="open" :close="close">
          <UButton v-bind="{ ...omit(ui.default, ['openIcon', 'closeIcon']), ...$attrs, ...omit(item, ['slot', 'disabled', 'content', 'defaultOpen']) }">
            <template #trailing>
              <UIcon
                :name="!open ? openIcon : closeIcon ? closeIcon : openIcon"
                class="ms-auto transform"
                :class="[
                  open && !closeIcon ? '-rotate-180' : '',
                  uiButton.icon.size[item.size || uiButton.default.size]
                ]"
              />
            </template>
          </UButton>
        </slot>
      </HDisclosureButton>

      <StateEmitter :open="open" @open="closeOthers(index)" />

      <Transition
        v-bind="ui.transition"
        @enter="onEnter"
        @after-enter="onAfterEnter"
        @before-leave="onBeforeLeave"
        @leave="onLeave"
      >
        <div v-show="open">
          <HDisclosurePanel :class="[ui.item.base, ui.item.size, ui.item.color, ui.item.padding]" static>
            <slot :name="item.slot || 'item'" :item="item" :index="index" :open="open" :close="close">
              {{ item.content }}
            </slot>
          </HDisclosurePanel>
        </div>
      </Transition>
    </HDisclosure>
  </div>
</template>

<script lang="ts">
import { defineComponent, computed } from 'vue'
import type { PropType } from 'vue'
import { Disclosure as HDisclosure, DisclosureButton as HDisclosureButton, DisclosurePanel as HDisclosurePanel } from '@headlessui/vue'
import { defu } from 'defu'
import { omit } from 'lodash-es'
import UIcon from '../elements/Icon.vue'
import UButton from '../elements/Button.vue'
import StateEmitter from '../../utils/StateEmitter'
import type { Button } from '../../types/button'
import { useAppConfig } from '#imports'
// TODO: Remove
// @ts-expect-error
import appConfig from '#build/app.config'

export default defineComponent({
  components: {
    HDisclosure,
    HDisclosureButton,
    HDisclosurePanel,
    UIcon,
    UButton,
    StateEmitter
  },
  inheritAttrs: false,
  props: {
    items: {
      type: Array as PropType<Partial<Button & { slot: string, disabled: boolean, content: string, defaultOpen: boolean, closeOthers: boolean }>[]>,
      default: () => []
    },
    defaultOpen: {
      type: Boolean,
      default: false
    },
    openIcon: {
      type: String,
      default: () => appConfig.ui.accordion.default.openIcon
    },
    closeIcon: {
      type: String,
      default: () => appConfig.ui.accordion.default.closeIcon
    },
    multiple: {
      type: Boolean,
      default: false
    },
    ui: {
      type: Object as PropType<Partial<typeof appConfig.ui.accordion>>,
      default: () => appConfig.ui.accordion
    }
  },
  setup (props) {
    // TODO: Remove
    const appConfig = useAppConfig()

    const ui = computed<Partial<typeof appConfig.ui.accordion>>(() => defu({}, props.ui, appConfig.ui.accordion))

    const uiButton = computed<Partial<typeof appConfig.ui.button>>(() => appConfig.ui.button)

    const buttonRefs = ref<Function[]>([])

    function closeOthers (itemIndex: number) {
      if (!props.items[itemIndex].closeOthers && props.multiple) {
        return
      }

      buttonRefs.value.forEach((close, index) => {
        if (index === itemIndex) return

        close()
      })
    }

    function onEnter (el: HTMLElement, done) {
      el.style.height = '0'
      el.offsetHeight // Trigger a reflow, flushing the CSS changes
      el.style.height = el.scrollHeight + 'px'

      el.addEventListener('transitionend', done, { once: true })
    }

    function onBeforeLeave (el: HTMLElement) {
      el.style.height = el.scrollHeight + 'px'
      el.offsetHeight // Trigger a reflow, flushing the CSS changes
    }

    function onAfterEnter (el: HTMLElement) {
      el.style.height = 'auto'
    }

    function onLeave (el: HTMLElement, done) {
      el.style.height = '0'

      el.addEventListener('transitionend', done, { once: true })
    }

    return {
      // eslint-disable-next-line vue/no-dupe-keys
      ui,
      uiButton,
      buttonRefs,
      closeOthers,
      omit,
      onEnter,
      onBeforeLeave,
      onAfterEnter,
      onLeave
    }
  }
})
</script>
