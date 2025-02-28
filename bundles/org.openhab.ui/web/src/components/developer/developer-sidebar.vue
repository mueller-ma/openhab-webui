<template>
  <f7-page class="developer-sidebar">
    <f7-navbar title="Developer Sidebar" subtitle="(Shift+Alt+D)" :color="$f7.data.themeOptions.dark === 'dark' ? '' : 'black'">
      <f7-subnavbar :inner="false" v-if="!$theme.md">
        <f7-searchbar custom-search placeholder="Search and Pin" :backdrop="false" @searchbar:search="search" @searchbar:clear="clearSearch" />
      </f7-subnavbar>
    </f7-navbar>
    <f7-subnavbar :inner="false" v-if="$theme.md">
      <f7-searchbar custom-search placeholder="Search and Pin" :backdrop="false" @searchbar:search="search" @searchbar:clear="clearSearch" />
    </f7-subnavbar>
    <div v-if="!searching" class="developer-sidebar-content">
      <f7-segmented strong tag="p" style="margin-right: calc(var(--f7-searchbar-inner-padding-right) + var(--f7-safe-area-right)); margin-left: calc(var(--f7-searchbar-inner-padding-left) + var(--f7-safe-area-left))">
        <f7-button :active="activeTab === 'pin'" icon-f7="pin_fill" icon-size="18" @click="activeTab = 'pin'" />
        <f7-button :active="activeTab === 'events'" icon-f7="bolt_horizontal_fill" icon-size="18" @click="activeTab = 'events'" />
        <f7-button :active="activeTab === 'scripting'" icon-f7="pencil_ellipsis_rectangle" icon-size="18" @click="activeTab = 'scripting'" />
        <f7-button :active="activeTab === 'tools'" icon-f7="rectangle_stack_badge_plus" icon-size="18" @click="activeTab = 'tools'" />
      </f7-segmented>
      <div v-if="activeTab === 'pin'">
        <f7-block class="no-margin no-padding">
          <f7-block-title class="padding-horizontal" medium>
            Pinned Objects
          </f7-block-title>
        </f7-block>
        <f7-block class="no-margin no-padding" v-if="!pinnedObjects.items.length && !pinnedObjects.things.length && !pinnedObjects.rules.length && !pinnedObjects.pages.length">
          <p class="padding-horizontal">
            Use the search box above or the button below to temporarily pin objects here for quick access.
          </p>
          <p class="padding-horizontal">
            <f7-button fill color="blue" @click="openModelPicker">
              Pin Items from Model
            </f7-button>
          </p>
        </f7-block>
        <f7-block class="no-margin no-padding" v-if="pinnedObjects.items.length">
          <f7-block-title class="padding-horizontal display-flex">
            <span>Pinned Items</span>
            <span style="margin-left:auto">
              <!-- <f7-link color="gray" icon-f7="eye" icon-size="14"></f7-link> -->
              <f7-link color="gray" icon-f7="list_bullet_indent" icon-size="14" @click="openModelPicker" />
              <f7-link color="gray" icon-f7="multiply" icon-size="14" @click="unpinAll('items')" />
            </span>
          </f7-block-title>
          <f7-list>
            <ul>
              <item v-for="item in pinnedObjects.items" :key="item.name" link="" :item="item" :context="context" :no-icon="true" :no-type="true" @click="(evt) => showItem(evt, item)">
                <div class="display-flex align-items-flex-end justify-content-flex-end" style="margin-top: 3px" slot="footer">
                  <f7-link class="margin-right itemlist-actions" color="gray" icon-f7="pencil" icon-size="18" tooltip="Edit" :href="'/settings/items/' + item.name" :animate="false" />
                  <f7-link class="itemlist-actions" color="red" icon-f7="pin_slash_fill" icon-size="18" tooltip="Unpin" @click="unpin('items', item, 'name')" />
                </div>
              </item>
            </ul>
            <!-- <f7-list-button title="Pick Items" @click="modelPickerOpened = true"></f7-list-button> -->
          </f7-list>
        </f7-block>
        <f7-block class="no-margin no-padding" v-if="pinnedObjects.things.length">
          <f7-block-title class="padding-horizontal display-flex">
            <span>Pinned Things</span>
            <span style="margin-left:auto">
              <f7-link color="gray" icon-f7="multiply" icon-size="14" @click="unpinAll('things')" />
            </span>
          </f7-block-title>
          <f7-list media-list>
            <ul>
              <f7-list-item v-for="thing in pinnedObjects.things" :key="thing.UID" media-item
                            :title="thing.label" :footer="thing.UID">
                <f7-badge slot="after" :color="thingStatusBadgeColor(thing.statusInfo)" :tooltip="thing.statusInfo.description">
                  {{ thingStatusBadgeText(thing.statusInfo) }}
                </f7-badge>
                <div class="display-flex align-items-flex-end justify-content-flex-end" style="margin-top: 3px" slot="footer">
                  <f7-link class="margin-right" :icon-color="(thing.statusInfo.statusDetail === 'DISABLED') ? 'orange' : 'gray'" :tooltip="(thing.statusInfo.statusDetail === 'DISABLED') ? 'Enable' : 'Disable'" icon-f7="pause_circle" icon-size="18" @click="toggleThingDisabled(thing)" />
                  <f7-link class="margin-right" color="gray" icon-f7="pencil" icon-size="18" tooltip="Edit" :href="'/settings/things/' + thing.UID" :animate="false" />
                  <f7-link color="red" icon-f7="pin_slash_fill" icon-size="18" tooltip="Unpin" @click="unpin('things', thing, 'UID')" />
                </div>
              </f7-list-item>
            </ul>
          </f7-list>
        </f7-block>
        <f7-block class="no-margin no-padding" v-if="pinnedObjects.rules.length">
          <f7-block-title class="padding-horizontal display-flex">
            <span>Pinned Rules</span>
            <span style="margin-left:auto">
              <f7-link color="gray" icon-f7="multiply" icon-size="14" @click="unpinAll('rules')" />
            </span>
          </f7-block-title>
          <f7-list media-list>
            <ul>
              <f7-list-item v-for="rule in pinnedObjects.rules" :key="rule.uid" media-item
                            :title="rule.name" :footer="rule.uid">
                <f7-badge slot="after" :color="ruleStatusBadgeColor(rule.status)" :tooltip="rule.status.description">
                  {{ ruleStatusBadgeText(rule.status) }}
                </f7-badge>
                <div class="display-flex align-items-flex-end justify-content-flex-end" style="margin-top: 3px" slot="footer">
                  <f7-link class="margin-right" :icon-color="(rule.status.statusDetail === 'DISABLED') ? 'orange' : 'gray'" :tooltip="(rule.status.statusDetail === 'DISABLED') ? 'Enable' : 'Disable'" icon-f7="pause_circle" icon-size="18" @click="toggleRuleDisabled(rule)" />
                  <f7-link class="margin-right" color="blue" icon-f7="play" icon-size="18" tooltip="Run" @click="runRuleNow(rule)" />
                  <f7-link class="margin-right" color="gray" icon-f7="pencil" icon-size="18" tooltip="Edit" :href="'/settings/' + (rule.tags.indexOf('Script') >= 0 ? 'scripts' : 'rules') + '/' + rule.uid" :animate="false" />
                  <f7-link color="red" icon-f7="pin_slash_fill" icon-size="18" tooltip="Unpin" @click="unpin('rules', rule, 'uid')" />
                </div>
              </f7-list-item>
            </ul>
          </f7-list>
        </f7-block>
        <f7-block class="no-margin no-padding" v-if="pinnedObjects.pages.length">
          <f7-block-title class="padding-horizontal display-flex">
            <span>Pinned Pages</span>
            <span style="margin-left:auto">
              <f7-link color="gray" icon-f7="multiply" icon-size="14" @click="unpinAll('pages')" />
            </span>
          </f7-block-title>
          <f7-list media-list>
            <ul>
              <f7-list-item v-for="page in pinnedObjects.pages" :key="page.uid" media-item
                            :title="page.config.label" :footer="page.uid">
                <div class="display-flex align-items-flex-end justify-content-flex-end" style="margin-top: 3px" slot="footer">
                  <!-- <f7-link class="margin-right" color="blue" icon-f7="rectangle_on_rectangle" icon-size="18" tooltip="Open in Popup" /> -->
                  <f7-link class="margin-right" color="blue" icon-f7="play" icon-size="18" tooltip="View" :href="'/page/' + page.uid" :animate="false" />
                  <f7-link class="margin-right" color="gray" icon-f7="pencil" icon-size="18" tooltip="Edit" :href="'/settings/pages/' + getPageType(page).type + '/' + page.uid" :animate="false" />
                  <f7-link color="red" icon-f7="pin_slash_fill" icon-size="18" tooltip="Unpin" @click="unpin('pages', page, 'uid')" />
                </div>
              </f7-list-item>
            </ul>
          </f7-list>
        </f7-block>
      </div>

      <div v-else-if="activeTab === 'events'">
        <f7-block class="no-margin no-padding">
          <f7-block-title class="padding-horizontal display-flex" medium>
            <span>Event Monitor</span>
            <span style="margin-left:auto">
              <f7-link :color="eventTopicFilter ? 'blue' : 'gray'" :icon-f7="eventTopicFilter ? 'line_horizontal_3_decrease_circle_fill' : 'line_horizontal_3_decrease_circle'" icon-size="14" tooltip="Filter topics" @click="changeEventTopicFilter" />
            </span>
          </f7-block-title>
          <f7-block>
            <p v-if="!sseClient">
              <f7-button fill color="blue" @click="startSSE">
                Stream Events
              </f7-button>
            </p>
            <p v-if="sseClient">
              <f7-button fill color="red" @click="stopSSE">
                Stop Streaming
              </f7-button>
            </p>
          </f7-block>
          <f7-list media-list>
            <f7-list-item v-for="event in sseEvents" :key="event.time.getTime()" :title="event.topic" :subtitle="event.type" :footer="event.payload" />
          </f7-list>
        </f7-block>
      </div>

      <div v-else-if="activeTab === 'scripting'">
        <f7-block class="no-margin no-padding">
          <f7-block-title class="padding-horizontal" medium>
            Code Tools
          </f7-block-title>
        </f7-block>
        <expression-tester />
        <f7-block class="no-margin no-padding">
          <f7-block-title class="padding-horizontal">
            Scripting Scratchpad
          </f7-block-title>
          <f7-list>
            <f7-list-button @click="openScriptingScratchpad" color="blue">
              Open Scratchpad
            </f7-list-button>
          </f7-list>
        </f7-block>
      </div>

      <div v-else-if="activeTab === 'tools'">
        <f7-block class="no-margin no-padding">
          <f7-block-title class="padding-horizontal" medium>
            Create Shortcuts
          </f7-block-title>
        </f7-block>
        <f7-block class="no-margin no-padding">
          <f7-list>
            <f7-list-item divider title="Things" />
            <f7-list-button href="/settings/things/add" color="blue" :animate="false">
              Add Thing
            </f7-list-button>
            <f7-list-button @click="quickAddThing" color="blue">
              Add Thing (quick)
            </f7-list-button>
            <f7-list-button href="/settings/things/inbox" color="blue" :animate="false">
              Inbox
            </f7-list-button>
            <f7-list-item divider title="Items" />
            <f7-list-button href="/settings/items/add" color="blue" :animate="false">
              Create Item
            </f7-list-button>
            <f7-list-button href="/settings/items/add-from-textual-definition" color="blue" :animate="false">
              Add Items (textual)
            </f7-list-button>
            <f7-list-item divider title="Pages" />
            <f7-list-button href="/settings/pages/layout/add" color="blue" :animate="false">
              Create layout page
            </f7-list-button>
            <f7-list-button href="/settings/pages/tabs/add" color="blue" :animate="false">
              Create tabbed page
            </f7-list-button>
            <f7-list-button href="/settings/pages/map/add" color="blue" :animate="false">
              Create map view
            </f7-list-button>
            <f7-list-button href="/settings/pages/plan/add" color="blue" :animate="false">
              Create floor plan
            </f7-list-button>
            <f7-list-button href="/settings/pages/chart/add" color="blue" :animate="false">
              Create chart
            </f7-list-button>
            <f7-list-button href="/settings/pages/sitemap/add" color="blue" :animate="false">
              Create sitemap
            </f7-list-button>
            <f7-list-item divider title="Automation" />
            <f7-list-button href="/settings/rules/add" color="blue" :animate="false">
              Create rule
            </f7-list-button>
            <f7-list-button href="/settings/scripts/add" color="blue" :animate="false">
              Create script
            </f7-list-button>
            <f7-list-button href="/settings/schedule/add" color="blue" :animate="false">
              Create scheduled rule
            </f7-list-button>
            <f7-list-item divider title="Advanced" />
            <f7-list-button href="/developer/widgets/add" color="blue" :animate="false">
              Create widget
            </f7-list-button>
            <f7-list-button href="/developer/blocks/add" color="blue" :animate="false">
              Create block library
            </f7-list-button>
          </f7-list>
        </f7-block>
      </div>
    </div>

    <f7-popover ref="itemPopover" class="item-popover">
      <item-standalone-control v-if="openedItem" :item="openedItem" :context="context" :no-border="true" />
    </f7-popover>
    <search-results v-if="searching" :searchResults="searchResults" :pinnedObjects="pinnedObjects" @pin="pin" @unpin="unpin" :cachedObjects="cachedObjects" :loading="searchResultsLoading" />
  </f7-page>
</template>

<style lang="stylus">
.panel-right.panel-in-breakpoint:before
  position absolute
  left 0
  top 0
  height 100%
  width 1px
  background rgba(0,0,0,0.1)
  content ''
  z-index 6000

.developer-sidebar
  scrollbar-width none /* Firefox */
  -ms-overflow-style none  /* IE 10+ */

  .developer-sidebar-content
    margin-top var(--f7-subnavbar-height)
    padding-top 0.3rem

  &.page
    background #e7e7e7 !important

  .page-content
    overflow-x hidden
.md .developer-sidebar-content
  margin-top 0
.theme-dark
  .developer-sidebar
    &.page
      background #232323 !important
</style>

<script>
import Item from '@/components/item/item.vue'
import ItemStandaloneControl from '@/components/item/item-standalone-control.vue'
import ModelPickerPopup from '@/components/model/model-picker-popup.vue'
import SearchResults from './search-results.vue'
import ExpressionTester from './expression-tester.vue'

import RuleStatus from '@/components/rule/rule-status-mixin'
import ThingStatus from '@/components/thing/thing-status-mixin'

export default {
  mixins: [RuleStatus, ThingStatus],
  components: {
    Item,
    ItemStandaloneControl,
    SearchResults,
    ExpressionTester
  },
  data () {
    return {
      searchQuery: '',
      searchResultsLoading: false,
      searching: false,
      activeTab: 'pin',
      monitoredItems: [],
      sseClient: null,
      eventTopicFilter: '',
      eventSource: null,
      cachedObjects: null,
      searchResults: {
        items: [],
        things: [],
        rules: [],
        pages: []
      },
      pinnedObjects: {
        items: [],
        things: [],
        rules: [],
        pages: []
      },
      sseEvents: [],
      openedItem: null,
      pageTypes: [
        { type: 'sitemap', label: 'Sitemap', componentType: 'Sitemap', icon: 'menu' },
        { type: 'layout', label: 'Layout', componentType: 'oh-layout-page', icon: 'rectangle_grid_2x2' },
        { type: 'home', label: 'Home', componentType: 'oh-home-page', icon: 'house' },
        { type: 'tabs', label: 'Tabbed', componentType: 'oh-tabs-page', icon: 'squares_below_rectangle' },
        { type: 'map', label: 'Map', componentType: 'oh-map-page', icon: 'map' },
        { type: 'plan', label: 'Floor plan', componentType: 'oh-plan-page', icon: 'square_stack_3d_up' },
        { type: 'chart', label: 'Chart', componentType: 'oh-chart-page', icon: 'graph_square' }
      ],
      testExpression: '',
      addThingAutocomplete: null
    }
  },
  computed: {
    context () {
      return {
        store: this.$store.getters.trackedItems
      }
    }
  },
  mounted () {
    this.startEventSource()
  },
  beforeDestroy () {
    this.stopEventSource()
    if (this.addThingAutocomplete) this.addThingAutocomplete.destroy()
  },
  methods: {
    itemContext (item) {
      return {

      }
    },
    addItemsFromModel (value) {
      this.$set(this.pinnedObjects, 'items', [...value])
    },
    openModelPicker () {
      const popup = {
        component: ModelPickerPopup
      }

      this.$f7.views.main.router.navigate({
        url: 'pick-from-model',
        route: {
          path: 'pick-from-model',
          popup
        }
      }, {
        props: {
          value: this.pinnedObjects.items,
          multiple: true,
          allowEmpty: true,
          popupTitle: 'Pin Items from Model',
          actionLabel: 'Pin'
        }
      })

      this.$f7.once('itemsPicked', this.addItemsFromModel)
      this.$f7.once('modelPickerClosed', () => {
        this.$f7.off('itemsPicked', this.addItemsFromModel)
      })
    },
    search (searchbar, query, previousQuery) {
      if (!query) {
        this.clearSearch()
        return
      }
      this.searching = true
      this.searchQuery = query

      if (this.searchResultsLoading) return

      const promises = (this.cachedObjects)
        ? [
          Promise.resolve(this.cachedObjects[0]),
          Promise.resolve(this.cachedObjects[1]),
          Promise.resolve(this.cachedObjects[2]),
          Promise.resolve(this.cachedObjects[3])
        ] : [
          this.$oh.api.get('/rest/items'),
          this.$oh.api.get('/rest/things?summary=true'),
          this.$oh.api.get('/rest/rules?summary=true'),
          Promise.resolve(this.$store.getters.pages)
        ]

      this.searchResultsLoading = true
      Promise.all(promises).then((data) => {
        this.$set(this, 'cachedObjects', data)
        this.searchResultsLoading = false
        const items = data[0].filter((i) => i.name.toLowerCase().indexOf(this.searchQuery.toLowerCase()) >= 0 || (i.label && i.label.toLowerCase().indexOf(this.searchQuery.toLowerCase()) >= 0))
        const things = data[1].filter((t) => t.UID.toLowerCase().indexOf(this.searchQuery.toLowerCase()) >= 0 || t.label.toLowerCase().indexOf(this.searchQuery.toLowerCase()) >= 0)
        const rules = data[2].filter((r) => r.uid.toLowerCase().indexOf(this.searchQuery.toLowerCase()) >= 0 || r.name.toLowerCase().indexOf(this.searchQuery.toLowerCase()) >= 0)
        const pages = data[3].filter((p) => p.uid.toLowerCase().indexOf(this.searchQuery.toLowerCase()) >= 0)
        this.$set(this, 'searchResults', {
          items,
          things,
          rules,
          pages
        })
      })
    },
    clearSearch () {
      this.searching = false
      this.searchResultsLoading = false
      this.searchSuery = ''
      this.$set(this, 'cachedObjects', null)
      this.$set(this, 'searchResults', { items: [], things: [], rules: [], pages: [] })
    },
    pin (type, obj) {
      this.pinnedObjects[type].push(obj)
    },
    unpin (type, obj, keyName) {
      let index = this.pinnedObjects[type].findIndex((o) => o[keyName] === obj[keyName])
      if (index >= 0) {
        this.pinnedObjects[type].splice(index, 1)
      }
    },
    unpinAll (type) {
      this.$set(this.pinnedObjects, type, [])
    },
    getPageType (page) {
      return this.pageTypes.find(t => t.componentType === page.component)
    },
    showItem (evt, item) {
      evt.cancelBubble = true
      if (this.$$(evt.target).closest('.itemlist-actions').length) return
      const itemEl = this.$$(evt.target).closest('.itemlist-item')
      if (!itemEl.length) return
      this.openedItem = item
      this.$nextTick(() => this.$refs.itemPopover.f7Popover.open(itemEl[0]))
    },
    toggleThingDisabled (thing) {
      const enable = (thing.statusInfo.statusDetail === 'DISABLED')
      this.$oh.api.putPlain('/rest/things/' + thing.UID + '/enable', enable.toString()).then((data) => {
        this.$f7.toast.create({
          text: (enable) ? 'Thing enabled' : 'Thing disabled',
          destroyOnClose: true,
          closeTimeout: 2000
        }).open()
      }).catch((err) => {
        this.$f7.toast.create({
          text: 'Error while disabling or enabling: ' + err,
          destroyOnClose: true,
          closeTimeout: 2000
        }).open()
      })
    },
    toggleRuleDisabled (rule) {
      const enable = (rule.status.statusDetail === 'DISABLED')
      this.$oh.api.postPlain('/rest/rules/' + rule.uid + '/enable', enable.toString()).then((data) => {
        this.$f7.toast.create({
          text: (enable) ? 'Rule enabled' : 'Rule disabled',
          destroyOnClose: true,
          closeTimeout: 2000
        }).open()
      }).catch((err) => {
        this.$f7.toast.create({
          text: 'Error while disabling or enabling: ' + err,
          destroyOnClose: true,
          closeTimeout: 2000
        }).open()
      })
    },
    runRuleNow (rule) {
      if (rule.status === 'RUNNING') return
      this.$f7.toast.create({
        text: 'Running rule',
        destroyOnClose: true,
        closeTimeout: 2000
      }).open()
      this.$oh.api.postPlain('/rest/rules/' + rule.uid + '/runnow', '').catch((err) => {
        this.$f7.toast.create({
          text: 'Error while running rule: ' + err,
          destroyOnClose: true,
          closeTimeout: 2000
        }).open()
      })
    },
    openScriptingScratchpad () {
      this.$oh.api.get('/rest/rules/scratchpad')
        .then((data) => {
          this.$f7.views.main.router.navigate('/settings/scripts/scratchpad', { animate: false })
        })
        .catch(() => {
          this.$oh.api.get('/rest/module-types/script.ScriptAction').then((data) => {
            const languages = data.configDescriptions.find((c) => c.name === 'type').options
            this.$f7.actions.create({
              buttons: [
                [
                  { label: true, text: 'Scripting Language' },
                  ...languages.map((l) => {
                    return {
                      text: l.label,
                      color: 'blue',
                      onClick: () => {
                        const scratchpad = {
                          uid: 'scratchpad',
                          name: '-Scratchpad-',
                          description: 'Created from the developer sidebar on ' + new Date().toLocaleDateString() + ' ' + new Date().toLocaleTimeString(),
                          triggers: [],
                          conditions: [],
                          actions: [
                            {
                              id: 'script',
                              type: 'script.ScriptAction',
                              configuration: {
                                type: l.value,
                                script: ''
                              }
                            }
                          ],
                          tags: ['Script', 'Scratchpad']
                        }
                        this.$oh.api.postPlain('/rest/rules', JSON.stringify(scratchpad), 'text/plain', 'application/json').then(() => {
                          this.$f7.toast.create({
                            text: 'Scratchpad script created',
                            destroyOnClose: true,
                            closeTimeout: 2000
                          }).open()
                          this.$f7.views.main.router.navigate('/settings/scripts/scratchpad', { animate: false })
                        })
                      }
                    }
                  })
                ],
                [
                  { color: 'red', text: 'Cancel', close: true }
                ]
              ]
            }).open()
          })
        })
    },
    quickAddThing () {
      if (this.addThingAutocomplete) {
        this.addThingAutocomplete.value = []
        this.addThingAutocomplete.open()
      } else {
        this.$f7.preloader.show()
        const self = this
        this.$oh.api.get('/rest/thing-types').then((data) => {
          const listedThingTypes = data.filter((t) => t.listed).map((t) => { return { UID: t.UID, label: `${t.label} (${t.UID})` } }).sort((a, b) => a.label.localeCompare(b.label))
          this.$f7.preloader.hide()
          this.addThingAutocomplete = this.$f7.autocomplete.create({
            openIn: 'popup',
            autoFocus: true,
            value: [],
            pageTitle: 'Select Thing Type',
            searchbarPlaceholder: 'Search thing types',
            requestSourceOnOpen: true,
            multiple: false,
            valueProperty: 'UID',
            textProperty: 'label',
            url: 'quick-add-thing/',
            source (query, render) {
              if (query.length === 0) {
                render(listedThingTypes)
              } else {
                render(listedThingTypes.filter((t) => (t.label.toLowerCase().indexOf(query.toLowerCase()) >= 0 || t.UID.toLowerCase().indexOf(query.toLowerCase()) >= 0)))
              }
            },
            on: {
              change (value) {
                if (!value.length) return
                self.$f7.views.main.router.navigate('/settings/things/add/' + value[0].UID.split(':')[0] + '/' + value[0].UID, { animate: false })
              }
            }
          }).open()
        })
      }
    },
    changeEventTopicFilter () {
      this.$f7.dialog.prompt('Filter events by topics (comma-separated, wildcards accepted):',
        'Event Monitor',
        (filter) => {
          this.eventTopicFilter = filter
        },
        null,
        this.eventTopicFilter)
    },
    startSSE () {
      this.$set(this, 'sseEvents', [])
      this.sseClient = this.$oh.sse.connect('/rest/events' + (this.eventTopicFilter ? '?topics=' + this.eventTopicFilter : ''), '', (event) => {
        event.time = new Date()
        this.sseEvents.unshift(...[event])
        this.sseEvents.splice(20)
      })
    },
    stopSSE () {
      this.$oh.sse.close(this.sseClient)
      this.sseClient = null
    },
    startEventSource () {
      this.eventSource = this.$oh.sse.connect('/rest/events?topics=openhab/rules/*/*,openhab/things/*/*,openhab/addons/*/*', null, (event) => {
        const topicParts = event.topic.split('/')
        switch (topicParts[1]) {
          case 'addons':
            if (this.addThingAutocomplete) this.addThingAutocomplete.destroy()
            break
          case 'things':
            switch (topicParts[3]) {
              case 'removed':
                this.unpin('things', { UID: topicParts[2] }, 'UID')
                break
              case 'status':
                const updatedThing = this.pinnedObjects.things.find((t) => t.UID === topicParts[2])
                if (!updatedThing) break
                const newStatus = JSON.parse(event.payload)
                if (updatedThing) {
                  if (updatedThing.statusInfo.status !== newStatus.status) updatedThing.statusInfo.status = newStatus.status
                  if (updatedThing.statusInfo.statusDetail !== newStatus.statusDetail) updatedThing.statusInfo.statusDetail = newStatus.statusDetail
                  if (updatedThing.statusInfo.description !== newStatus.description) updatedThing.statusInfo.description = newStatus.description
                }
            }
            break
          case 'rules':
            switch (topicParts[3]) {
              case 'removed':
                this.unpin('rules', { uid: topicParts[2] }, 'uid')
                break
              case 'state':
                const rule = this.pinnedObjects.rules.find((r) => r.uid === topicParts[2])
                if (!rule) break
                this.$set(rule, 'status', JSON.parse(event.payload))
            }
            break
        }
      })
    },
    stopEventSource () {
      this.$oh.sse.close(this.eventSource)
      this.eventSource = null
    }
  }
}
</script>
