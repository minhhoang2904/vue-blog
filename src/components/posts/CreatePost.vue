<template>
  <div class="relative">
    <!-- Floating Action Button -->
    <button 
      @click="toggleCreate"
      class="fixed bottom-6 right-6 w-14 h-14 bg-black text-white rounded-full shadow-lg hover:shadow-xl hover:scale-110 transition-all duration-300 z-50 flex items-center justify-center text-2xl border-2 border-white"
      :class="{ 'rotate-45': isCreateOpen }"
    >
      {{ isCreateOpen ? '×' : '+' }}
    </button>

    <!-- Slide-down Create Panel -->
    <div 
      class="fixed inset-x-0 top-0 bg-white border-b-2 border-blog-primary shadow-2xl transition-transform duration-500 z-50 h-1/3"
      :class="isCreateOpen ? 'transform translate-y-0' : 'transform -translate-y-full'"
    >
      <div class="max-w-2xl mx-auto p-6 h-full flex flex-col">
        <h2 class="text-xl font-bold mb-4 text-center">Create New Post</h2>
        
        <!-- Post Type Selector -->
        <div class="flex gap-2 mb-4 justify-center">
          <button 
            v-for="type in postTypes" 
            :key="type.id"
            @click="selectedType = type.id"
            class="px-4 py-2 rounded-full border-2 transition-all duration-200"
            :class="selectedType === type.id 
              ? 'bg-blog-primary text-white border-blog-primary' 
              : 'border-blog-primary hover:bg-gray-100'"
          >
            {{ type.icon }} {{ type.name }}
          </button>
        </div>

        <!-- Content Input -->
        <textarea 
          v-model="postContent"
          placeholder="What's on your mind?"
          class="w-full h-32 p-4 border-2 border-blog-primary rounded-lg resize-none focus:outline-none focus:ring-4 focus:ring-blog-accent focus:ring-opacity-30 focus:border-blog-accent transition-all duration-200 min-h-0"
        ></textarea>

        <!-- Action Buttons -->
        <div class="flex justify-between items-center mt-4">
          <div class="flex gap-2">
            <button class="p-2 rounded-full border-2 border-blog-primary hover:bg-blog-primary hover:text-white transition-all duration-200">
              📷
            </button>
            <button class="p-2 rounded-full border-2 border-blog-primary hover:bg-blog-primary hover:text-white transition-all duration-200">
              🎵
            </button>
            <button class="p-2 rounded-full border-2 border-blog-primary hover:bg-blog-primary hover:text-white transition-all duration-200">
              📍
            </button>
          </div>
          
          <div class="flex gap-2">
            <button 
              @click="toggleCreate"
              class="px-6 py-2 border-2 border-gray-300 rounded-full hover:bg-gray-100 transition-all duration-200"
            >
              Cancel
            </button>
            <button 
              @click="createPost"
              class="px-6 py-2 bg-black text-white rounded-full hover:from-blue-600 hover:to-purple-700 hover:scale-105 transition-all duration-200 font-semibold shadow-md"
            >
              Post
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Backdrop -->
    <div 
      v-if="isCreateOpen"
      @click="toggleCreate"
      class="fixed inset-0 bg-black bg-opacity-50 transition-opacity duration-300 z-30"
    ></div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const isCreateOpen = ref(false)
const selectedType = ref('text')
const postContent = ref('')

const postTypes = [
  { id: 'text', name: 'Text', icon: '📝' },
  { id: 'photo', name: 'Photo', icon: '📸' },
  { id: 'video', name: 'Video', icon: '🎥' },
  { id: 'music', name: 'Music', icon: '🎵' }
]

const toggleCreate = () => {
  isCreateOpen.value = !isCreateOpen.value
}

const createPost = () => {
  // Add post creation logic here
  console.log('Creating post:', { type: selectedType.value, content: postContent.value })
  postContent.value = ''
  toggleCreate()
}
</script>

<style scoped>
/* Custom animations can be added here if needed */
</style>
