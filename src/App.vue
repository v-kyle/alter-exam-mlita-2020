<template>
    <v-app>
        <v-app-bar
                app
                color="primary"
                dark
                dense
        >
            <v-toolbar-title class="mx-auto">FOXRAM</v-toolbar-title>
            <v-dialog
                    v-model="dialog"
                    width="960"
            >
                <v-card>
                    <v-card-title
                            class="headline lighten-2"
                            primary-title
                    >
                        МЛиТА. Автоматизацией доказательств в исчислении высказываний
                    </v-card-title>

                    <v-card-text>
                        Данный сайт - альтернативный экзамен по предмету МЛиТА студентов группы 8307 - Никулина Л. и Зуба В.
                        <br/>
                        На сайте представлены модули для ознакомления и изучения темы "Автоматизацией доказательств в исчислении высказываний".
                        <br/>
                        Перед началом работы, настоятельно рекомендуется ознакомиться с <a target="_blank" href="https://docs.google.com/presentation/d/1QWKZYDRfYKyDRM2LjOALzYinpYVWN9bKQeTi9DNP7Oc/edit?usp=sharing"> презентацией данного веб-приложения.</a>
                        <br/>
                        Удачи в изучении математики!
                    </v-card-text>

                    <v-divider/>

                    <v-card-actions>
                        <v-spacer/>
                        <v-btn color="primary" @click="dialog = false">
                            Перейти на сайт
                        </v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
        </v-app-bar>

        <v-content>
            <v-container class="fill-height" fluid>
                <v-row align="center" justify="center" no-gutters>
                    <v-col class="text-center" cols="12" sm="10" md="8">
                        <v-card raised>
                            <v-tabs
                                    v-model="tab"
                                    background-color="transparent"
                                    color="basil"
                                    grow
                            >
                                <v-tab
                                        v-for="(module, index) in modules"
                                        :key="index"
                                >
                                    <h4>{{ module.title }}</h4>
                                </v-tab>
                            </v-tabs>

                            <v-card-title>
                                {{ modules[tab].title }}
                            </v-card-title>

                            <v-card-text class="text-left" v-html="instruction"/>

                            <v-card-subtitle class="text-left">
                                <b>Краткое описание:</b>
                                {{ modules[tab].description }}
                            </v-card-subtitle>

                            <v-card-text>
                                <v-tabs-items v-model="tab">
                                    <v-tab-item v-for="(module, index) in modules" :key="index">
                                        <module0 v-if="index===0"/>
                                        <module1 v-if="index===1"/>
                                        <module2 v-if="index===2"/>
                                        <module3 v-if="index===3"/>
                                        <module3up v-if="index===4"/>
                                        <sentences v-if="index===5"/>
                                    </v-tab-item>
                                </v-tabs-items>
                            </v-card-text>
                        </v-card>
                    </v-col>
                </v-row>
            </v-container>
        </v-content>


        <v-footer
                color="primary"
                app
        >
            <v-row justify="center" align="center">
                <v-col class="white--text text-center">МЛИТА 2020</v-col>
            </v-row>
        </v-footer>
    </v-app>
</template>

<script>
    import module0 from "./components/module0";
    import module1 from "./components/module1";
    import module2 from "./components/module2";
    import module3 from "./components/module3";
    import module3up from "./components/module3up";
    import sentences from "./components/sentences";

    export default {
        name: 'App',

        components: {
            module0,
            module1,
            module2,
            module3,
            module3up,
            sentences
        },

        data: () => ({
            dialog: true,
            modules: [
                {
                    title: 'Общезначимость',
                    description: 'Модуль для проверки формулы на общезначимость (то есть при любых значениях аргументов результат - истина)'
                },
                {
                    title: 'Противоречивость',
                    description: 'Модуль для проверки формулы на противоречивость (то есть при любых значениях аргументов результат - ложь)'
                },
                {
                    title: 'Основная теорема логического вывода',
                    description: 'Использование формулы F1∧F2∧..∧Fn→G и дальнейшая проверка на общезначимость'},
                {
                    title: 'Метод резолюций',
                    description: 'Использование метода резолюции для проверки истинности высказывания'
                },
                {
                    title: 'Метод резолюций: "вычеркивание"',
                    description: 'Использование метода резолюции с использованием метода вычеркивания для более оптимальных вычислений'
                },
                // {
                //     title: 'Метод резолюций: "человеческий вид"',
                //     description: 'Посылки вводятся в виде предложений на русском языке, все посылки должны соответствовать строгим критерии'
                // }

            ],
            operations: [
                {operation: 'Отрицание', sign: '!'},
                {operation: 'Конъюнкция', sign: '∧'},
                {operation: 'Дизъюнкция', sign: '∨'},
                {operation: 'Импликация', sign: '→'},
                {operation: 'Эквиваленция', sign: '⇔'},
            ],
            tab: 0,
            currentModule: 0,
            instruction: "<ul> <li><b>!</b> - Отрицание; <b>∧</b> - Конъюнкция; <b>∨</b> - Дизъюнкция; <b>→</b> - Импликация; <b>⇔</b> - Эквивалентность;<br/></li>  <li>Также, если возможность вводить знаки при помощи клавиатуры: <b>*</b> - Конъюнкция; <b>+</b> - Дизъюнкция; <b>-></b> - Импликация; <b><=></b> - Эквивалентность; <br/></li> <li>Между символами требуется ПРОБЕЛ.</li> </ul>"
        }),
        computed: {},
        methods: {},
    };
</script>
