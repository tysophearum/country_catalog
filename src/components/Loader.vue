<script>
export default {
    data() {
        return {
            num: Array.from({ length: 25 }, (_, index) => index + 1)
        }
    }
}
</script>
<template>
  <div class=" wrapper fixed left-0 top-0 right-0 bottom-0 bg-gray-300/50 z-50 flex items-center justify-center">
    <span class="loading loading-spinner loading-lg"></span>
  </div>
</template>