<template>
    <div class="col s12 m6">
        <div>
            <div class="page-subtitle">
                <h4>{{'Edit' | localize}}</h4>
            </div>

            <form @submit.prevent="submitHandler">
                <div class="input-field" >
                    <select ref="select" v-model="current">
                        <option
                            v-for="c of categories"
                            :key="c.id"
                            :value="c.id"
                        >{{c.title}}</option>
                    </select>
                    <label>{{'ChooseCategory' | localize}}</label>
                </div>

                <div class="input-field">
                    <input
                            type="text"
                            id="name"
                            v-model="title"
                            :class="{invalid: $v.title.$dirty && !$v.title.required}"
                    >
                    <label for="name">{{'Title' | localize}}</label>
                    <span
                            v-if="$v.title.$dirty && !$v.title.required"
                            class="helper-text invalid"
                    >
                        {{'Message_EnterCategoryTitle' | localize}}
                    </span>
                </div>

                <div class="input-field">
                    <input
                            id="limit"
                            type="number"
                            v-model="limit"
                            :class="{invalid: $v.limit.$dirty && !$v.title.minValue}"
                    >
                    <label for="limit">{{'Limit' | localize}}</label>
                    <span
                            v-if="$v.limit.$dirty && !$v.title.minValue"
                            class="helper-text invalid"
                    >
                        {{'Message_MinimalValue' | localize}}: {{$v.limit.$params.minValue.min}}</span>
                </div>

                <button class="btn waves-effect waves-light" type="submit">
                    {{'Refresh' | localize}}
                    <i class="material-icons right">send</i>
                </button>
            </form>
        </div>
    </div>
</template>

<script>
    import {required, minValue} from 'vuelidate/lib/validators'

    export default {
        name: "CategoryEdit",
        props: {
            categories: {
                type: Array,
                required: true
            }
        },
        data: () => ({
            select: null,
            title: '',
            limit: 1,
            current: null
        }),
        watch: {
            current(catId) {
                const {title, limit} = this.categories.find(c => c.id === catId)
                this.title = title
                this.limit = limit
            }
        },
        validations: {
            title: {required},
            limit: {minValue: minValue(1)}
        },
        methods: {
            async submitHandler() {
                if (this.$v.$invalid) {
                    this.$v.$touch()
                    return
                }

                try {
                    const categoryData = {
                        id: this.current,
                        title: this.title,
                        limit: this.limit
                    }
                    await this.$store.dispatch('updateCategory', categoryData)
                    this.$message('Категория успешно обновлена')
                    this.$emit('updated', categoryData)
                } catch (e) {

                }
            }
        },
        created() {
            const {id, title, limit} = this.categories[0]
            this.current = id
            this.title = title
            this.limit = limit
        },
        mounted() {


            M.updateTextFields()
            this.select = M.FormSelect.init(this.$refs.select)
        },
        destroyed() {
            if (this.select && this.select.destroy) {
                this.select.destroy
            }
        }
    }
</script>

<style scoped>

</style>