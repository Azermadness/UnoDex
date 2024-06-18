<script setup>
    import { ref, computed } from "vue";
    let pokemons = ref([]);
    let types = ref([]);
    let moves = ref([]);
    let search = ref("");
    let detail_id = ref();
    let detail_bool = ref();
    let enabled = ref();
    let filters_bool = ref();
    let bool_and = ref();
    let bool_or = ref();

    fetch("https://web24.mmi-stdie.fr/nathan/wp-json/truc/pokemon")
        .then((response) => {
            response.json().then((data) => {
                pokemons.value = data;
                pokemons.value.sort(function (a, b) {
                    return a.acf.pokemon_id - b.acf.pokemon_id;
                });
            });
        })
        .catch((error) => {
            console.log(error);
        });

    fetch("https://web24.mmi-stdie.fr/nathan/wp-json/truc/move")
        .then((response) => {
            response.json().then((data) => {
                moves.value = data;
            });
        })
        .catch((error) => {
            console.log(error);
        });

    fetch("https://web24.mmi-stdie.fr/nathan/wp-json/truc/types")
        .then((response) => {
            response.json().then((data) => {
                types.value = data;
            });
        })
        .catch((error) => {
            console.log(error);
        });

    const filteredPokemons = computed(() => {
        return pokemons.value.filter((pokemon) =>
            pokemon.acf.pokemon_name.includes(toCamelCaseWord(search.value))
        );
    });

    const getMoves = computed(() => {
        return moves.value;
    });

    const getTypes = computed(() => {
        return types.value;
    });

    function returnTypes(pkmn) {
        let temp = [];
        let temp2 = [];

        /* try {
            typeof pkmn.acf.pokemon_type[1].ID;
            temp2 = [pkmn.acf.pokemon_type[0].ID, pkmn.acf.pokemon_type[1].ID];
        } catch {
            try {
                typeof pkmn.acf.pokemon_type[0].ID;
                temp2 = [pkmn.acf.pokemon_type[0].ID];
            } catch (err) {
                temp2 = [];
            }
        } */

        pkmn.acf.pokemon_type.forEach((elem) => {
            temp2.push(elem.ID);
        });

        types.value.forEach((elem) => {
            temp2.forEach((elem2) => {
                if (elem.ID == elem2) {
                    temp.push([elem.acf.type_name, elem.acf.type_color]);
                }
            });
        });

        return temp;
    }

    function returnMoves(pkmn) {
        let tempMovesID = [];
        let tempType;
        let tempMoves = [];

        pkmn.acf.pokemon_move.forEach((elem) => {
            tempMovesID.push(elem.ID);
        });

        tempMovesID.forEach((elem) => {
            console.log(elem);
            moves.value.forEach((elem2) => {
                console.log(elem2.ID);
                if (elem == elem2.ID) {
                    console.log("hit");
                    tempType = elem2.acf.move_type[0].ID;
                    types.value.forEach((type) => {
                        tempType = tempType == type.ID ? type : tempType;
                    });
                    tempMoves.push([
                        elem2.acf.pkmn_move_name,
                        tempType.acf.type_name,
                        tempType.acf.type_color,
                        elem2.acf.move_power,
                    ]);
                }
            });
        });
        console.log(tempMoves);
        return tempMoves;
    }

    /* function returnMoves(pkmn, i, tempMovesID) {
        if (tempMovesID[0] == 0) {
            tempMovesID = [];
            i = 0;
            console.log("1st move");
        } else {
            console.log("moves recalled" + i);
        }

        try {
            tempMovesID.push(pkmn.acf.pokemon_move[i].ID);
            returnMoves(pkmn, i + 1, tempMovesID);
            console.log("move added" + i);
        } catch (error) {
            let tempType = [];
            let tempMoves = [];
            console.log(error);
            console.log("total moves : " + i);
            tempMovesID.forEach((elem) => {
                console.log(elem);
                moves.value.forEach((elem2) => {
                    console.log(elem2.ID);
                    if (elem == elem2.ID) {
                        console.log("hit");
                        tempType = elem2.acf.move_type[0].ID;
                        types.value.forEach((type) => {
                            tempType = tempType == type.ID ? type : tempType;
                        });
                        tempMoves.push([
                            elem2.acf.pkmn_move_name,
                            tempType.acf.type_name,
                            tempType.acf.type_color,
                            elem2.acf.move_power,
                        ]);
                    }
                });
            });
            console.log(tempMoves);
            return tempMoves;
        }
    } */

    function toCamelCaseWord(str) {
        // only upper case the first char and lowercase the rest.
        return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
    }
</script>

<template>
    <header>
        <div id="close_wrapper" v-if="enabled" @click="enabled = false"></div>
        <nav>
            <div
                :id="[enabled ? 'pkb_on' : 'pkb_off']"
                @click="enabled = !enabled"
            >
                <div id="pkb_top"></div>
                <div :id="[enabled ? 'pkb_bot_on' : 'pkb_bot_off']"></div>
            </div>
            <div :id="[enabled ? 'nav_bg_on' : 'nav_bg_off']" class="nav_bg">
                <input
                    type="text"
                    id="searchbar"
                    placeholder="Search..."
                    v-model="search"
                    @change="
                        detail_bool = search != '' ? false : detail_bool;
                        filters_bool = search != '' ? false : filters_bool;
                    "
                />
                <div
                    id="b_filters"
                    @click="
                        detail_bool = false;
                        filters_bool = true;
                    "
                >
                    <div id="b_filters_logo"></div>
                    Filters
                </div>
                <div id="socials">
                    <div id="rs_discord"></div>
                    <div id="rs_reddit"></div>
                    <div id="rs_github"></div>
                </div>
            </div>
            <div
                id="homepage"
                @click="
                    detail_bool = false;
                    filters_bool = false;
                    search = '';
                "
            >
                UnoDex
            </div>
        </nav>
    </header>
    <main>
        <div></div>
        <section id="pokemon_list" v-if="!detail_bool && !filters_bool">
            <article
                v-for="(pkmn, index) in filteredPokemons"
                :key="index"
                @click="
                    detail_id = pkmn.acf.pokemon_id;
                    detail_bool = true;
                "
            >
                <div class="pkmn_item">
                    <div class="pkb_icon"></div>
                    <div class="pkmn_sprite">
                        <img
                            :src="
                                'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/shiny/' +
                                pkmn.acf.pokemon_id +
                                '.png'
                            "
                        />
                    </div>
                    <div class="pkmn_text">
                        <div>{{ pkmn.acf.pokemon_name }}</div>
                        <div class="list_type_wrapper">
                            <div
                                class="list_type_bg"
                                v-for="single_type in returnTypes(pkmn)"
                                :style="
                                    'background-color:' + single_type[1] + ';'
                                "
                            >
                                {{ single_type[0] }}
                            </div>
                        </div>
                    </div>
                    <div class="pkmn_number">#{{ pkmn.acf.pokemon_id }}</div>
                </div>
            </article>
        </section>
        <section
            id="pokemon_detail"
            v-for="(pkmn, index) in filteredPokemons"
            v-if="detail_bool"
            :index="index"
        >
            <article id="single_view" v-if="pkmn.acf.pokemon_id == detail_id">
                <div class="single_view_w1">
                    <div class="single_img_id">
                        <div class="pkmn_sprite_single">
                            <img
                                :src="
                                    'https://www.pokencyclopedia.info/sprites/gen5/ani_black-white_shiny/ani_bw-S_' +
                                    pkmn.acf.pokemon_id +
                                    '.gif'
                                "
                            />
                        </div>
                        <div>#{{ pkmn.acf.pokemon_id }}</div>
                    </div>
                    <div class="single_types">
                        <div
                            class="single_type_bg"
                            v-for="single_type in returnTypes(pkmn)"
                            :style="'background-color:' + single_type[1] + ';'"
                        >
                            {{ single_type[0] }}
                        </div>
                    </div>
                    <div class="single_name">{{ pkmn.acf.pokemon_name }}</div>
                </div>

                <div class="single_view_w2">
                    <div class="single_moves_title">Moves Learned</div>
                    <div
                        v-for="(move, mIndex) in returnMoves(pkmn)"
                        :index="mIndex"
                        class="single_move_wrapper"
                    >
                        <div class="single_move_type">
                            <div
                                class="single_move_type_bg"
                                :style="'background-color:' + move[2] + ';'"
                            >
                                {{ move[1] }}
                            </div>
                        </div>
                        <div class="single_move_name">{{ move[0] }}</div>
                        <div class="single_move_bp">{{ move[3] }}</div>
                    </div>
                </div>
            </article>
        </section>
        <section v-if="filters_bool">
            <div id="filter_view">
                <div id="filter_view_title">Type Filter</div>
                <div id="filter_button_flex">
                    <div
                        class="filter_button"
                        :style="
                            'background-color: ' +
                            (bool_and ? 'var(--def_type)' : 'var(--main)') +
                            ';'
                        "
                        id="button_and"
                        @click="
                            bool_and = true;
                            bool_or = false;
                        "
                    >
                        AND
                    </div>
                    <div
                        class="filter_button"
                        :style="
                            'background-color: ' +
                            (bool_or ? 'var(--def_type)' : 'var(--main)') +
                            ';'
                        "
                        id="button_or"
                        @click="
                            bool_and = false;
                            bool_or = true;
                        "
                    >
                        OR
                    </div>
                </div>
                <div id="filter_types">
                    <div
                        v-for="(type, tIndex) in getTypes"
                        class="filter_single_type"
                        :style="'background-color:' + type.acf.type_color + ';'"
                    >
                        {{ type.acf.type_name }}
                    </div>
                </div>
            </div>
        </section>
    </main>
</template>

<style scoped></style>
