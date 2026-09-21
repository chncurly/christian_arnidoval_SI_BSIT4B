<template>
  <v-layout class="rounded rounded-md" style="min-height: 100vh;">
    <v-main class="d-flex align-center justify-center">
      <v-container class="fill-height d-flex align-center justify-center">
      </v-container>
    </v-main>
  </v-layout>
</template>

<script setup lang="ts">
//@ts-nocheck
const user = ref<any>(null)

onMounted(() => {
  const savedUser = localStorage.getItem('google_user')

  if (savedUser) {
    user.value = JSON.parse(savedUser)
  } else {
    // Awtomatikong ipapadala sa /login kapag walang naka-save na user
    navigateTo('/login')
  }
})

const logout = () => {
  localStorage.removeItem('google_user')
  localStorage.removeItem('google_token')
  navigateTo('/login')
}
</script>