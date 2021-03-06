<template>
    <div>
        <section class="section content">
            <div class="container has-text-centered" v-if="isLoading">
                <h1 class="title is-3">Loading...</h1>

                <p class="button is-loading transparent"></p>
            </div>

            <div class="container has-text-centered" v-else-if="isError">
                <h4 class="title is-4">Found no matching server with an ID of {{ this.$route.params.id }}</h4>
            </div>

            <div class="container has-text-centered" v-else-if="!enabled">
                <h1 class="title is-1">{{ name }}</h1>
                <h4 class="title is-4">This server does not have the leveling feature enabled :(</h4>
            </div>

            <div class="container has-text-centered" v-else>

                <div class="columns">
                    <div class="column">
                        <h1 class="title is-1">{{ name }}</h1>
                        <p>Showing the top 100 players on the <strong>{{ name }}</strong> leaderboard.</p>
                    </div>
                </div>

                <div class="columns">
                    <div class="column">
                        <table class="table is-striped is-fullwidth is-hoverable">
                            <tbody>
                                <player-experience-table-row
                                    v-for="user of leaderboardUsers"
                                    :key="user.id"
                                    :serverModifier="modifier"
                                    :user="user"
                                />
                            </tbody>
                        </table>
                    </div>
                </div>

                <div class="columns">
                    <div class="column has-text-centered">
                        <p class="update-counter">Automatically updating the leaderboard in <strong>{{ updatesIn }}</strong> seconds.</p>
                    </div>
                </div>

            </div>
        </section>

        <footer class="footer">
            <div class="content has-text-centered">
                <p>
                    Created by <a href="https://senither.com/">Alexis Tan</a>, powered by <a href="https://bulma.io/">Bulma</a> and <a href="https://vuejs.org/">VueJS</a>.
                    <br>Get the <a href="https://github.com/avaire/leaderboard">source code</a> on <a href="https://github.com/avaire/leaderboard">GitHub</a>.
                </p>
            </div>
        </footer>
    </div>
</template>

<script>
    import config from '../config';
    import PlayerExperienceTableRow from './PlayerExperienceTableRow';

    export default {
        mounted() {
            this.updateLeaderboard();

            setInterval(() => {
                if (--this.updatesIn <= 0) {
                    this.updateLeaderboard();
                    this.updatesIn = config.updateRate;
                }
            }, 1000);
        },
        data() {
            return {
                name: null,
                enabled: false,
                modifier: -1,
                leaderboard: [],
                isLoading: true,
                isError: false,
                updatesIn: config.updateRate,
            };
        },
        computed: {
            leaderboardUsers() {
                let index = 1;
                let leaderboardUsers = [];
                for (let user of this.leaderboard) {
                    user.rank = index++;
                    leaderboardUsers.push(user);
                }
                return leaderboardUsers;
            }
        },
        methods: {
            updateLeaderboard() {
                let endpoint = config.interalApi;
                if (endpoint.substr(endpoint.length - 1, 1) == '/') {
                    endpoint = endpoint.substr(0, endpoint.length - 1);
                }

                axios(`${endpoint}/leaderboard/${config.serverId}`).then(response => {
                    if (response.status == 200) {
                        this.name = response.data.name;
                        this.enabled = response.data.enabled;
                        this.modifier = response.data.modifier;
                        this.leaderboard = response.data.leaderboard;
                    }

                    this.isLoading = false;
                }).catch(error => this.$router.push({ name: 'home' }));
            },
        },
        components: {
            PlayerExperienceTableRow
        }
    }
</script>
