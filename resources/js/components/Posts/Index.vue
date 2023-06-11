<script setup>
import { onMounted, ref, watch } from "vue";
import { Bootstrap5Pagination } from "laravel-vue-pagination";

const posts = ref({
    data: [],
    isLoading: true,
});

const categories = ref({});
const category_id = ref("");
const sort_field = ref("created_at");
const sort_order = ref("desc");

// Methods
const getCategories = async () => {
    const response = await axios.get(`/api/categories`);
    categories.value = response.data.data;
};

const getPosts = async (page = 1) => {
    posts.value.isLoading = true;
    const response = await axios.get(`/api/posts`, {
        params: {
            page,
            category_id: category_id.value,
            sort_field: sort_field.value,
            sort_order: sort_order.value,
        },
    });
    posts.value.data = response.data;
    posts.value.isLoading = false;
};

const handleSort = (field) => {
    if (sort_field.value == field) {
        sort_order.value = sort_order.value == "asc" ? "desc" : "asc";
    } else {
        sort_order.value = "asc";
        sort_field.value = field;
    }
    getPosts();
};

watch(
    () => category_id.value,
    (newVal, prevVal) => {
        getPosts();
    }
);

// Hooks
onMounted(async () => {
    await getCategories();
    await getPosts();
});
</script>
<template>
    <div class="container mt-5">
        <div class="row mb-4">
            <div class="col">
                <select v-model="category_id" class="form-select">
                    <option value="">--Choose Category--</option>
                    <option
                        v-for="category in categories"
                        :key="category.id"
                        :value="category.id"
                    >
                        {{ category.name }}
                    </option>
                </select>
            </div>
            <div class="col"></div>
            <div class="col"></div>
        </div>
        <div class="row">
            <div class="col">
                <div v-if="!posts.isLoading">
                    <table class="table">
                        <thead>
                            <tr>
                                <th scope="col">ID</th>
                                <th scope="col">
                                    <a
                                        href="#"
                                        @click.prevent="handleSort('title')"
                                    >
                                        Title
                                    </a>
                                    <span
                                        v-if="
                                            sort_field == 'title' &&
                                            sort_order == 'asc'
                                        "
                                        >&uparrow;</span
                                    >
                                    <span
                                        v-if="
                                            sort_field == 'title' &&
                                            sort_order == 'desc'
                                        "
                                        >&darr;</span
                                    >
                                </th>
                                <th scope="col">
                                    <a
                                        href="#"
                                        @click.prevent="handleSort('body')"
                                    >
                                        Body
                                    </a>
                                    <span
                                        v-if="
                                            sort_field == 'body' &&
                                            sort_order == 'asc'
                                        "
                                        >&uparrow;</span
                                    >
                                    <span
                                        v-if="
                                            sort_field == 'body' &&
                                            sort_order == 'desc'
                                        "
                                        >&darr;</span
                                    >
                                </th>
                                <th scope="col">
                                    <a
                                        href="#"
                                        @click.prevent="
                                            handleSort('created_at')
                                        "
                                    >
                                        Publish Date
                                    </a>
                                    <span
                                        v-if="
                                            sort_field == 'created_at' &&
                                            sort_order == 'asc'
                                        "
                                        >&uparrow;</span
                                    >
                                    <span
                                        v-if="
                                            sort_field == 'created_at' &&
                                            sort_order == 'desc'
                                        "
                                        >&darr;</span
                                    >
                                </th>
                                <th scope="col">Action</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="post in posts.data.data" :key="post.id">
                                <td>{{ post.id }}</td>
                                <td>{{ post.title }}</td>
                                <td>{{ post.body.substring(0, 50) }}</td>
                                <td>{{ post.created_at }}</td>
                            </tr>
                        </tbody>
                    </table>
                    <Bootstrap5Pagination
                        :data="posts.data"
                        @pagination-change-page="getPosts"
                    />
                </div>
                <div v-else class="text-center mt-5">Loading...</div>
            </div>
        </div>
    </div>
</template>
