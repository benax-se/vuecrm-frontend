<template>
    <nav class="navbar teal">
        <div class="nav-wrapper">
            <div class="navbar-left">
                <a href="#" @click.prevent="$emit('click')">
                    <i class="material-icons black-text">dehaze</i>
                </a>
                <span class="black-text">{{date | date('datetime')}}</span>
            </div>



            <ul class="right hide-on-small-and-down" @click="">
                <div class="switch">
                    <label>
                        EN
                        <input type="checkbox" v-model="isRuLocale">
                        <span class="lever"></span>
                        RU
                    </label>
                </div>
                <li>

                    <a
                            class="dropdown-trigger black-text"
                            href="#"
                            data-target="dropdown"
                            ref="dropdown"
                    >
                        {{name}}
                        <i class="material-icons right">arrow_drop_down</i>
                    </a>

                    <ul id='dropdown' class='dropdown-content'>
                        <li>
                            <router-link to="/profile" class="black-text">
                                <i class="material-icons">account_circle</i>{{'Profile' | localize}}
                            </router-link>
                        </li>
                        <li class="divider" tabindex="-1"></li>
                        <li>
                            <a href="#" class="black-text" @click.prevent="logout">
                                <i class="material-icons">assignment_return</i>{{'Logout' | localize}}
                            </a>
                        </li>
                    </ul>
                </li>
            </ul>
        </div>
    </nav>
</template>

<script>
    export default {
        name: "Navbar",
        data: () => ({
            date: new Date(),
            interval: null,
            dropdown: null,
            isRuLocale: true,
        }),
        computed: {
            name() {
                return this.$store.getters.info.name
            }
        },
        watch: {
            async isRuLocale(locale) {
                try {
                    await this.$store.dispatch('updateInfo', {
                        locale: this.isRuLocale ? 'ru-RU' : 'en-US'
                    })
                } catch (e) {

                }
            }
        },
        methods: {
            async logout() {
                await this.$store.dispatch('logout')
                this.$router.push('/login?message=logout')
            }
        },
        mounted() {
            this.interval = setInterval(() => {
                this.date = new Date()
            }, 1000)
            this.isRuLocale = this.$store.getters.info.locale === 'ru-RU'
            this.dropdown = M.Dropdown.init(this.$refs.dropdown, {
                constrainWidth: true,
                coverTrigger: false
            })
        },
        beforeDestroy() {
            clearInterval(this.interval)
            if (this.dropdown && this.dropdown.destroy) {
                this.dropdown.destroy()
            }
        }

    }
</script>

<style scoped>
    .switch {
        display: inline-flex;
        margin-left: 1rem;
    }
    label {
        color: #000 !important;
        font-size: 0.9rem;
    }
</style>