<script>
import SmoothScrollbar from "smooth-scrollbar"
import { h } from "vue"

import _ from "./helpers"

const defaultOptions = {
  damping: 0.1,
  thumbMinSize: 20,
  renderByPixels: true,
  alwaysShowTracks: false,
  continuousScrolling: false,
  delegateTo: null,
  plugins: {},
}

const Event = {
  loading: "loading",
  endY: "endy",
  endX: "endX",
  scroll: "scroll",
}

export default {
  name: "c-scroll-view",
  props: {
    infiniteLoading: {
      type: Boolean,
      default: false,
    },
    loadThreshold: {
      type: Number,
      default: 50,
    },
    options: {
      type: Object,
      default: () => ({}),
    },
    plugins: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      resolve: true,
      loading: false,
      completed: false,
      scrollBar: null,
      listeners: [],
      meta: {
        limit: {},
        offset: {},
      },
    }
  },
  computed: {
    hasPlugins() {
      if (_.isArray(this.plugins)) {
        return !!this.plugins.length
      } else {
        return false
      }
    },
  },
  methods: {
    /**
     * @param {String} axis - x or y axis
     * @return {Object}
     */
    getLimit(axis = "") {
      return _.getScrollState(this.scrollBar, axis, "limit")
    },

    /**
     * @param {String} axis - x or y axis
     * @return {Object}
     */
    getOffset(axis = "") {
      return _.getScrollState(this.scrollBar, axis, "offset")
    },

    // Smooth-scrollbar api methods
    /**
     *
     * @param {Number} x
     * @param {Number} y
     * @param {Number} duration
     * @param {Object} options
     * @see https://github.com/idiotWu/smooth-scrollbar/blob/develop/docs/api.md#scrollbarscrollto
     */
    scrollTo(x = 0, y = 0, duration = 300, options = {}) {
      this.scrollBar.scrollTo(x, y, duration, options)
    },

    /**
     *
     * @param {HTMLElement} elem
     * @param {Object} [options]
     * @see https://github.com/idiotWu/smooth-scrollbar/blob/develop/docs/api.md#scrollbarscrollintoview
     */
    scrollIntoView(elem, options = {}) {
      this.scrollBar.scrollIntoView(elem, options)
    },

    /**
     *
     * @param {HTMLElement} elem
     * @see https://github.com/idiotWu/smooth-scrollbar/blob/develop/docs/api.md#scrollbarisvisible
     */
    isVisible(elem) {
      this.scrollBar.isVisible(elem)
    },

    /**
     *
     * @param {Function} listener
     * @see https://github.com/idiotWu/smooth-scrollbar/blob/develop/docs/api.md#scrollbaraddlistener
     */
    addListener(listener) {
      this.listeners.push(listener)
      this.scrollBar.addListener(listener)
    },

    /**
     *
     * @param {Function} listener
     * @see https://github.com/idiotWu/smooth-scrollbar/blob/develop/docs/api.md#scrollbarremovelistener
     */
    removeListener(listener) {
      this.listeners = this.listeners.filter((attached) => {
        if (attached === listener) {
          this.scrollBar.removeListener(listener)
        }

        return attached !== listener
      })
    },

    /**
     *
     */
    removeAllListeners() {
      this.listeners.forEach((listener) => {
        this.scrollBar.removeListener(listener)
      })
      this.listeners = []
    },

    /**
     * @see https://github.com/idiotWu/smooth-scrollbar/blob/develop/docs/api.md#scrollbarupdate
     */
    update() {
      this.scrollBar.update()
    },

    // Infinite loading api methods

    /**
     * Emits loading event
     */
    debounceLoad: _.debounce(function () {
      if (this.resolve) {
        this.resolve = false
        this.loading = true

        this.emitLoad()
      }
    }, 300),

    /**
     * Emits loading event
     */
    emitLoad() {
      this.$emit(Event.loading, {
        loaded: () => this.setLoaded(),
        completed: () => this.setCompleted(),
      })
    },

    /**
     * Sets loaded state
     */
    setLoaded() {
      this.resolve = true
      this.loading = false
      this.completed = false

      this.$nextTick(() => {
        let limitY = this.getLimit("y")
        let offsetY = this.getOffset("y")

        if (_.checkLoadCapability(limitY, offsetY, this.loadThreshold)) {
          this.emitLoad()
        }
      })
    },

    /**
     * Sets completed state
     */
    setCompleted() {
      this.resolve = false
      this.loading = false
      this.completed = true
    },

    /**
     * Resets state
     */
    resetInfLoad() {
      this.resolve = true
      this.loading = false
      this.completed = false
    },

    // Misc
    focus() {
      this.scrollBar.containerEl.focus()
    },
    blur() {
      this.scrollBar.containerEl.blur()
    },
  },
  mounted() {
    this.$nextTick(() => {
      // Use plugins
      if (this.hasPlugins) {
        this.plugins.forEach((plugin) => {
          SmoothScrollbar.use(plugin)
        })
      }

      // Init
      this.scrollBar = SmoothScrollbar.init(
        this.$refs.view,
        _.defaultsDeep(this.options, defaultOptions)
      )

      // Add infinite loading listener
      this.addListener((status) => {
        if (!this.infiniteLoading) return
        if (this.loading || this.completed) return

        let { limit, offset } = status

        let canLoad = _.checkLoadCapability(
          limit.y,
          offset.y,
          this.loadThreshold
        )
        this.resolve = canLoad

        if (!this.completed) {
          if (canLoad) {
            this.debounceLoad(true)
          }
        } else {
          this.loading = false
        }
      })

      // Add scroll listener
      this.addListener((status) => {
        let { limit, offset } = status

        let limitX = limit.x
        let limitY = limit.y

        let offsetX = offset.x
        let offsetY = offset.y

        if (limitY > 0) {
          if (limitY === offsetY) {
            this.$emit(Event.endY)
          }
        }

        if (limitX > 0) {
          if (limitX === offsetX) {
            this.$emit(Event.endX)
          }
        }

        this.meta.limit = limit
        this.meta.offset = offset
        this.$emit(Event.scroll, status)
      })

      // Emit initial
      if (this.infiniteLoading) {
        this.emitLoad()
      }
    })
  },
  beforeUnmount() {
    if (this.scrollBar !== null) {
      this.removeAllListeners()
      this.scrollBar.destroy()
    }
  },
  updated() {
    this.scrollBar && this.scrollBar.update()
  },
  render() {
    let containerData = {
      class: "c-scroll-view",
      ref: "view",
      attrs: {
        "data-scrollbar": "",
      },
      on: {
        mouseenter: this.focus,
      },
      style: {
        display: "block",
        width: "100%",
        height: "100%",
      },
    }

    return h("div", containerData, [
      h(
        "div",
        {
          class: "c-scroll-view__content",
          style: {
            position: "relative",
            display: "block",
            height: "auto",
          },
        },
        this.$slots.default()
      ),
    ])
  },
}
</script>
