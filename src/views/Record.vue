<template>
    <div>
        <div class="page-title">
            <h3>{{'NewRecord' | localize}}</h3>
        </div>

        <Loader v-if="loading"/>

        <p class="center" v-else-if="!categories.length"> {{'Message_NoCategories' | localize}} <router-link to="/categories">
            {{'Message_AddNewCategory' | localize}}</router-link></p>

        <form class="form" v-else @submit.prevent="submitHandler">
            <div class="input-field" >
                <select ref="select" v-model="category">
                    <option
                        v-for="c of categories"
                        :key="c.id"
                        :value="c.id"
                    >{{c.title}}</option>
                </select>
                <label>{{'ChooseCategory' | localize}}</label>
            </div>

            <p>
                <label>
                    <input
                            class="with-gap"
                            name="type"
                            type="radio"
                            value="income"
                            v-model="type"
                    />
                    <span>{{'Income' | localize}}</span>
                </label>
            </p>

            <p>
                <label>
                    <input
                            class="with-gap"
                            name="type"
                            type="radio"
                            value="outcome"
                            v-model="type"
                    />
                    <span>{{'Outcome' | localize}}</span>
                </label>
            </p>

            <div class="input-field">
                <input
                        id="amount"
                        type="number"
                        v-model.number="amount"
                        :class="{invalid: $v.amount.$dirty && !$v.amount.minValue}"
                >
                <label for="amount">{{'Amount' | localize}}</label>
                <span class="helper-text invalid"
                      v-if="$v.amount.$dirty && !$v.amount.minValue"
                >{{'Message_MinimalAmount' | localize}}: {{$v.amount.$params.minValue.min}}</span>
            </div>

            <div class="input-field">
                <input
                        id="description"
                        type="text"
                        v-model="description"
                        :class="{invalid: $v.description.$dirty && !$v.description.required}"
                >
                <label for="description">{{'Description' | localize}}</label>
                <span
                        v-if="$v.description.$dirty && !$v.description.required"
                        class="helper-text invalid"
                >{{'Message_EnterDescription' | localize}}</span>
            </div>

            <button class="btn waves-effect waves-light" type="submit">
                {{'Create' | localize}}
                <i class="material-icons right">send</i>
            </button>
        </form>
    </div>
</template>

<script>
    import {minValue, required} from "vuelidate/lib/validators";
    import {mapGetters} from 'vuex'

    export default {
        name: "Record",
        metaInfo() {
            return {
                title: this.$title('Menu_NewRecord')
            }
        },
        data: () => ({
            loading: true,
            categories: [],
            category: null,
            type: 'income',
            amount: 1,
            description: ''
        }),
        computed: {
            ...mapGetters(['info']),
            canCreateRecord() {
                if (this.type === 'income'){
                    return true
                }

                return this.info.bill >= this.amount
            }
        },
        validations: {
            description: {required},
            amount: {minValue: minValue(1)}
        },
        async mounted() {
            this.categories = await this.$store.dispatch('fetchCategories')
            this.loading = false

            if (this.categories.length) {
                this.category = this.categories[0].id
            }

            setTimeout(()=> {
                this.select = M.FormSelect.init(this.$refs.select)
                M.updateTextFields()
            },0)
        },
        methods: {
            async submitHandler() {
                if (this.$v.$invalid) {
                    this.$v.$touch()
                    return
                }

                if (this.canCreateRecord) {
                    try {
                        await this.$store.dispatch('createRecord', {
                            categoryId: this.category,
                            amount: this.amount,
                            description: this.description,
                            type: this.type,
                            date: new Date().toJSON()
                        })

                        const bill = this.type === 'income'
                            ? this.info.bill + this.amount
                            : this.info.bill - this.amount

                        await this.$store.dispatch('updateInfo', {bill})
                        this.$message('Запись успешно создана')
                        this.$v.$reset()
                        this.amount=1
                        this.description=''
                    } catch (e) {
                        console.log(e)
                    }

                } else {
                    this.$message(`Недостаточно средства на счёте (не хватает ${this.amount - this.info.bill})`)
                }
            }
        }
    }
</script>

<style scoped>

</style>