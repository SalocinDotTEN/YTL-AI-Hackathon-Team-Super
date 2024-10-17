<template>
  <v-app>
    <v-main>
      <v-app-bar class="px-md-4" color="surface-variant" flat>
        <template #prepend>
          <v-app-bar-nav-icon v-if="$vuetify.display.smAndDown" @click="drawer = !drawer" />
        </template>
        <v-img class="me-sm-8" max-width="40" src="https://cdn.vuetifyjs.com/docs/images/logos/v.svg" />
        <template v-if="$vuetify.display.mdAndUp">
          <v-btn
            v-for="(item, i) in items"
            :key="i"
            :active="i === 0"
            class="me-2 text-none"
            slim
            v-bind="item"
          />
        </template>
        <v-spacer />
        <template #append>
          <v-btn class="ms-1 opacity-60" icon="mdi-bell-outline" />
          <v-btn class="ms-1" icon>
            <v-avatar image="https://picsum.photos/268" />
            <v-menu activator="parent" origin="top">
              <v-list>
                <v-list-item link title="Update profile" />
                <v-list-item link title="Sign out" />
              </v-list>
            </v-menu>
          </v-btn>
        </template>
      </v-app-bar>
      <v-navigation-drawer
        v-if="$vuetify.display.smAndDown"
        v-model="drawer"
        location="top"
        temporary
        width="355"
      >
        <v-list class="py-0" slim>
          <v-list-item
            v-for="(item, i) in items"
            :key="i"
            :active="i === 0"
            link
            :prepend-icon="item.icon"
            :title="item.text"
          />
        </v-list>
      </v-navigation-drawer>
      <v-toolbar color="surface" elevation="1">
        <template #title>
          <h2 class="text-h4 font-weight-bold">Document Comparer</h2>
        </template>
      </v-toolbar>
      <div class="pa-4">
        <v-sheet border="md" rounded="lg" width="100%">
          <router-view />
        </v-sheet>
      </div>
      <AppFooter />
    </v-main></v-app>
</template>

<script lang="ts" setup>
  import { shallowRef } from 'vue'

  const drawer = shallowRef(false)

  const items = [
    { text: 'Document Comparer' },
    { text: 'Knowledgebase' },
    { text: 'Calendar' },
    { text: 'Analytics' },
    { text: 'Inbox' },
    { text: 'Notifications' },
  ]
</script>
