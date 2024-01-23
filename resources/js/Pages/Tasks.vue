<script setup>
import ActionMessage from "@/Components/ActionMessage.vue";
import DangerButton from "@/Components/DangerButton.vue";
import DialogModal from "@/Components/DialogModal.vue";
import FormSection from "@/Components/FormSection.vue";
import InputError from "@/Components/InputError.vue";
import InputLabel from "@/Components/InputLabel.vue";
import PrimaryButton from "@/Components/PrimaryButton.vue";
import SecondaryButton from "@/Components/SecondaryButton.vue";
import SectionTitle from "@/Components/SectionTitle.vue";
import TextInput from "@/Components/TextInput.vue";
import AppLayout from "@/Layouts/AppLayout.vue";
import { useForm as usePrecognitionForm } from "laravel-precognition-vue-inertia";
import { useForm } from "@inertiajs/vue3";
import { ref } from "vue";

const props = defineProps(["tasks"]);

const formCreateTask = usePrecognitionForm("post", route("tasks.store"), {
    name: "",
});

formCreateTask.setValidationTimeout(300);

const createTask = () => {
    formCreateTask.submit({
        preserveScroll: true,
        onSuccess: () => {
            formCreateTask.name = "";
        },
    });
};

const formUpdateTask = useForm("put", {
    is_done: false,
});

const updateTask = (id, isDone) => {
    formUpdateTask.is_done = isDone;
    formUpdateTask.put(route("tasks.update", id), {
        url: route("tasks.update", id),
        preserveScroll: true,
        onSuccess: () => {
            //
        },
    });
};

const confirmingTaskDeletion = ref(false);
const taskIdToDelete = ref(null);
const formDeleteTask = useForm("delete", {});

const confirmTaskDeletion = (id) => {
    taskIdToDelete.value = id;
    confirmingTaskDeletion.value = true;
};

const deleteTask = () => {
    formDeleteTask.delete(route("tasks.destroy", taskIdToDelete.value), {
        preserveScroll: true,
        onSuccess: () => {
            confirmingTaskDeletion.value = false;
        },
    });
};

const closeModal = () => {
    confirmingTaskDeletion.value = false;
};
</script>

<template>
    <AppLayout title="Liste des tâches">
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Liste des tâches
            </h2>
        </template>

        <div class="max-w-7xl mx-auto py-10 sm:px-6 lg:px-8 space-y-12">
            <FormSection @submitted="createTask">
                <template #title> Nouvelle tâche </template>

                <template #description>
                    Ajouter une nouvelle tâche à la liste
                </template>

                <template #form>
                    <!-- Name -->
                    <div class="col-span-6 sm:col-span-4">
                        <InputLabel for="name" value="Nom de la tâche" />
                        <TextInput
                            id="name"
                            v-model="formCreateTask.name"
                            type="text"
                            class="mt-1 block w-full"
                            @input="formCreateTask.validate('name')"
                        />
                        <InputError
                            :message="formCreateTask.errors.name"
                            class="mt-2"
                        />
                    </div>
                </template>

                <template #actions>
                    <ActionMessage
                        :on="formCreateTask.recentlySuccessful"
                        class="me-3"
                    >
                        Saved.
                    </ActionMessage>

                    <PrimaryButton
                        :class="{ 'opacity-25': formCreateTask.processing }"
                        :disabled="formCreateTask.processing"
                    >
                        Save
                    </PrimaryButton>
                </template>
            </FormSection>

            <div class="md:grid md:grid-cols-3 md:gap-6 space-y-4 md:space-y-0">
                <SectionTitle>
                    <template #title> Mes tâches </template>
                    <template #description>
                        Marquez vos tâches comme terminées ou supprimez-les.
                    </template>
                </SectionTitle>

                <ul
                    class="grid grid-cols-1 divide-y bg-white shadow sm:rounded-md md:col-span-2"
                >
                    <li
                        class="flex space-x-4 items-center p-4 sm:p-6"
                        v-for="task in tasks"
                        :key="task.id"
                    >
                        <input
                            class="rounded text-indigo-500 outline-none focus:ring-2 focus:ring-indigo-500 cursor-pointer"
                            type="checkbox"
                            :id="task.id"
                            :value="task.id"
                            :checked="task.is_done"
                            @change="updateTask(task.id, $event.target.checked)"
                        />
                        <label class="cursor-pointer" :for="task.id">{{
                            task.name
                        }}</label>

                        <div class="flex-1 text-right">
                            <button
                                class="text-xl hover:scale-110 transition-all"
                                @click="confirmTaskDeletion(task.id)"
                            >
                                🚮
                            </button>
                        </div>
                    </li>
                </ul>
            </div>
        </div>
    </AppLayout>

    <DialogModal :show="confirmingTaskDeletion" @close="closeModal">
        <template #title> Supprimer la tâche </template>

        <template #content>
            Êtes-vous sûr de vouloir supprimer cette tâche ? Cette action est
            irréversible.
        </template>

        <template #footer>
            <SecondaryButton @click="closeModal"> Annuler </SecondaryButton>

            <DangerButton
                class="ms-3"
                :class="{ 'opacity-25': confirmingTaskDeletion.processing }"
                :disabled="confirmingTaskDeletion.processing"
                @click="deleteTask"
            >
                Supprimer
            </DangerButton>
        </template>
    </DialogModal>
</template>
