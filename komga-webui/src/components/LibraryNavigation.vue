<template>
  <div>
    <v-bottom-navigation
      v-if="show && bottomNavigation"
      grow color="primary"
      :fixed="bottomNavigation"
    >
      <v-btn v-if="showRecommended"
             :to="{name: 'recommended-libraries', params: {libraryId: libraryId}}"
      >
        <span>{{ $t('library_navigation.recommended') }}</span>
        <v-icon>mdi-star</v-icon>
      </v-btn>

      <v-btn :to="{name: 'browse-libraries', params: {libraryId: libraryId}}">
        <span>{{ $t('library_navigation.browse') }}</span>
        <v-icon>mdi-bookshelf</v-icon>
      </v-btn>

      <v-btn
        v-if="collectionsCount > 0"
        :to="{name: 'browse-collections', params: {libraryId: libraryId}}"
      >
        <span>{{ $t('library_navigation.collections') }}</span>
        <v-icon>mdi-layers-triple</v-icon>
      </v-btn>

      <v-btn
        v-if="readListsCount > 0"
        :to="{name: 'browse-readlists', params: {libraryId: libraryId}}"
      >
        <span>{{ $t('library_navigation.readlists') }}</span>
        <v-icon>mdi-book-multiple</v-icon>
      </v-btn>

    </v-bottom-navigation>

    <template
      v-if="show && !bottomNavigation"
    >
      <v-btn v-if="showRecommended"
             :to="{name: 'recommended-libraries', params: {libraryId: libraryId}}"
             text
             class="mx-1"
      >
        {{ $t('library_navigation.recommended') }}
      </v-btn>

      <v-btn :to="{name: 'browse-libraries', params: {libraryId: libraryId}}"
             text
             class="mx-1"
      >
        {{ $t('library_navigation.browse') }}
      </v-btn>

      <v-btn
        v-if="collectionsCount > 0"
        :to="{name: 'browse-collections', params: {libraryId: libraryId}}"
        text
        class="mx-1"
      >
        {{ $t('library_navigation.collections') }}
      </v-btn>

      <v-btn
        v-if="readListsCount > 0"
        :to="{name: 'browse-readlists', params: {libraryId: libraryId}}"
        text
        class="mx-1"
      >
        {{ $t('library_navigation.readlists') }}
      </v-btn>

    </template>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import {COLLECTION_ADDED, COLLECTION_DELETED, READLIST_ADDED, READLIST_DELETED} from '@/types/events'
import {LIBRARIES_ALL} from '@/types/library'

export default Vue.extend({
  name: 'LibraryNavigation',
  data: () => {
    return {
      collectionsCount: 0,
      readListsCount: 0,
    }
  },
  props: {
    libraryId: {
      type: String,
      required: true,
    },
    bottomNavigation: {
      type: Boolean,
      default: false,
    },
  },
  watch: {
    libraryId: {
      handler(val) {
        this.loadReadListCounts(val)
        this.loadCollectionCounts(val)
      },
      immediate: true,
    },
  },
  created() {
    this.$eventHub.$on(COLLECTION_ADDED, this.collectionAdded)
    this.$eventHub.$on(COLLECTION_DELETED, this.collectionDeleted)
    this.$eventHub.$on(READLIST_ADDED, this.readListAdded)
    this.$eventHub.$on(READLIST_DELETED, this.readListDeleted)
  },
  beforeDestroy() {
    this.$eventHub.$off(COLLECTION_ADDED, this.collectionAdded)
    this.$eventHub.$off(COLLECTION_DELETED, this.collectionDeleted)
    this.$eventHub.$off(READLIST_ADDED, this.readListAdded)
    this.$eventHub.$off(READLIST_DELETED, this.readListDeleted)
  },
  computed: {
    showRecommended(): boolean {
      return this.libraryId !== LIBRARIES_ALL
    },
    show(): boolean {
      return this.collectionsCount > 0 || this.readListsCount > 0 || this.showRecommended
    },
  },
  methods: {
    readListAdded() {
      if(this.readListsCount === 0) this.loadReadListCounts(this.libraryId)
    },
    readListDeleted() {
      if(this.readListsCount === 1) this.loadReadListCounts(this.libraryId)
    },
    collectionAdded() {
      if(this.collectionsCount === 0) this.loadCollectionCounts(this.libraryId)
    },
    collectionDeleted() {
      if(this.collectionsCount === 1) this.loadCollectionCounts(this.libraryId)
    },
    async loadCollectionCounts(libraryId: string) {
      const lib = libraryId !== LIBRARIES_ALL ? [libraryId] : undefined
      this.$komgaCollections.getCollections(lib, {size: 0})
      .then(v => this.collectionsCount = v.totalElements)
    },
    async loadReadListCounts(libraryId: string) {
      const lib = libraryId !== LIBRARIES_ALL ? [libraryId] : undefined
      await this.$komgaReadLists.getReadLists(lib, {size: 0})
        .then(v => this.readListsCount = v.totalElements)
    },
  },
})
</script>

<style scoped>

</style>
