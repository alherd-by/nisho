<template>
    <div class="scene">
        <router-link :to="{name: 'terms'}">
            <el-button>Вярнуцца</el-button>
        </router-link>
        <h3 class="pdng-t-15px">Дадаць слова</h3>
        <el-form :model="new_term"
                 ref="form"
                 :rules="rules"
                 @submit.prevent
                 label-width="120px"
                 label-position="top"
                 class="pdng-t-30px">
            <el-form-item label="Слова" required prop="term_name">
                <el-input v-model="new_term.term_name"/>
            </el-form-item>
            <el-form-item label="Вызначэнне" required prop="definition">
                <el-input v-model="new_term.definition"
                          :type="'textarea'"
                          :rows="7"
                          placeholder="Вызначэнне"/>
            </el-form-item>
            <el-form-item label="Прыклад" prop="example">
                <el-input v-model="new_term.example"
                          :type="'textarea'"
                          :rows="7"
                          placeholder="Прыклад"/>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="submit(form)">Дадаць</el-button>
                <router-link :to="{name: 'terms'}" class="mrgn-l-20px">
                    <el-button>Адмена</el-button>
                </router-link>
            </el-form-item>
        </el-form>
    </div>
</template>

<script setup>
import {reactive, ref} from 'vue'
import {supabase}      from "./supabase.js";
import {ElMessage}     from "element-plus";
import {useRouter}     from 'vue-router'
import {getUser}       from "./user.js";

const router = useRouter();

const new_term = reactive({
    term_name : '',
    definition: '',
    example   : '',
})
const form     = ref()
const rules    = reactive({
    term_name : [
        {required: true, message: 'Абавязкова', trigger: 'blur'},
        {min: 3, message: 'мінімум 3 сымбаля', trigger: 'blur'},
    ],
    definition: [
        {required: true, message: 'Абавязкова', trigger: 'blur'},
        {min: 10, message: 'мінімум 10 сымбалей', trigger: 'blur'},
    ],
})
const account  = ref(getUser());
const submit   = async (formEl) => {
    if (!formEl) {
        return
    }
    if (!account.value) {
        ElMessage.warning('Каб дадаць слова, вам трэба залагініцца')
        return;
    }
    await formEl.validate(async (valid) => {
        if (!valid) {
            return;
        }
        try {
            let {data, error} = await supabase.rpc(
                'add_term',
                {
                    definition: new_term.definition,
                    example   : new_term.example,
                    term_name : new_term.term_name.trim()
                }
            )
            if (error) {
                throw error
            }
            ElMessage.success('Паспяхова даданы тэрмін');
            await router.push({name: 'term', params: {id: data}})
        } catch (error) {
            ElMessage.error('Адбылася памылка')
            throw error;
        }
    })
}

</script>

<style scoped>

</style>
