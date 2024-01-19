<script setup>
import AppLayout from "@/Layouts/AppLayout.vue";

const props = defineProps(["tasks"]);
</script>

<template>
    <AppLayout title="Liste des tâches">
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Liste des tâches
            </h2>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                    <div v-for="task in tasks" :key="task.id">
                        {{ task.name }}
                    </div>
                </div>
            </div>
        </div>
    </AppLayout>
</template>
