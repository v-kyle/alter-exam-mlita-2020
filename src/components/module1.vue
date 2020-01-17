<template>
    <div>
        <v-form v-model="valid">
            <v-container>
                <v-text-field
                        ref="formInput"
                        v-model="input"
                        :rules="rules"
                        label="Formula"
                        required
                />
                <v-row>
                    <template v-for="(operation, index) in operations">
                        <v-col :key="index">
                            <v-btn @click="insertSpecialSymbol(index)">{{ operation.sign }}</v-btn>
                        </v-col>
                    </template>
                </v-row>
                <v-btn color="primary white--text" @click="getAnswer" :disabled="!valid">
                    Проверить
                </v-btn>
            </v-container>
        </v-form>
        <v-divider class="mx-4"/>
        <v-progress-linear :active="loading" :indeterminate="loading"/>
        <v-alert color="success" dense class="white--text">
            Результат: <br/> <b>{{ answer }}</b>
        </v-alert>
        <v-data-table :headers="headers" :items="items"/>
    </div>
</template>

<script>
    export default {
        name: "module1",
        data() {
            return {
                loading: false,
                input: '',
                answer: null,
                rules: [
                    value => !!value || 'Required.',
                ],
                operations: [
                    {operation: 'Отрицание', sign: '!'},
                    {operation: 'Конъюнкция', sign: '∧'},
                    {operation: 'Дизъюнкция', sign: '∨'},
                    {operation: 'Импликация', sign: '→'},
                    {operation: 'Эквиваленция', sign: '⇔'},
                ],
                valid: false,
                headers: [],
                items: [],
            }
        },
        methods: {
            convertSymbols(str) {
                str = str.replace(/\*/g, "∧");
                str = str.replace(/\+/g, "∨");
                str = str.replace(/->/g, "→");
                str = str.replace(/<=>/g, "⇔");
                return str.toUpperCase();
            },

            insertSpecialSymbol(signIndex) {
                this.input += this.operations[signIndex].sign;
                this.$refs.formInput.focus();
            },

            async getAnswer() {
                this.loading = true;
                let input = this.input;

                try {
                    let responce = await fetch(
                        'http://83.166.240.14:8080/api/inconsistency', {
                            method: 'POST',
                            headers: {
                                'Content-Type': 'application/json'
                            },
                            body: JSON.stringify({data: input})
                        });
                    let json = await responce.json();
                    if (!json.error) {
                        this.answer = (json.result)? 'Противоречива' : 'Не противоречива';

                        this.headers = Object.keys(json.description).map(el => {
                            return {text: el, value: el};
                        });

                        let matrix = this.headers.map((el)=>{
                            return json.description[el.text];
                        });

                        const transpose = matrix => matrix[0].map((col, i) => matrix.map(row => row[i]));

                        let transMatrix = transpose(matrix);

                        let headers = this.headers;

                        this.items = transMatrix.map(row=>{
                            return Object.assign({}, ...row.map((el, index) => {
                                return {[headers[index].text]: el};
                            }));
                        });
                    }
                } catch (e) {
                    // eslint-disable-next-line no-console
                    console.log(e);
                    this.answer = "Ошибка ввода!";
                } finally {
                    this.loading = false;
                }
            }
        },
        watch: {
            input(value) {
                this.input = this.convertSymbols(value);
            },
        }
    }
</script>

