<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-50">
    <div class="max-w-md w-full text-center">
      <h1 class="text-6xl font-bold text-gray-900 mb-4">403</h1>
      <h2 class="text-2xl font-semibold text-gray-700 mb-4">
        Unauthorized Access
      </h2>
      <p class="text-gray-600 mb-8">
        You don't have permission to access this page.
      </p>
      <NuxtLink
        to="/"
        class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        Go Back to Dashboard
      </NuxtLink>
    </div>
  </div>
</template>

<script setup>
definePageMeta({
  layout: false,
});
</script>
