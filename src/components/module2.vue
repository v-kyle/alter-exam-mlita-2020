<template>
    <div>
        <v-form v-model="valid">
            <v-container>
                <div><b>Введите массив формул (посылок):</b></div>
                <v-text-field
                        v-for="(input, index) in inputs"
                        :key="index"
                        ref="forms"
                        v-model="inputs[index]"
                        @click="changeActiveForm(index)"
                        :rules="rules"
                        label="Formula"
                        required
                />
                <v-btn @click="addInput">Добавить поле</v-btn>
                <v-divider class="my-4"/>
                <div><b>Предполагаемое следствие:</b></div>
                <v-text-field
                        ref="formula"
                        v-model="formula"
                        @click="changeActiveForm(-123)"
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
                    Подтвердить
                </v-btn>
            </v-container>
        </v-form>
        <v-progress-linear :active="loading" :indeterminate="loading"/>
        <v-alert color="success" dense class="white--text">
            Результат: <br/> <b>{{ answer }}</b>
        </v-alert>
        <v-data-table :headers="headers" :items="items"/>
    </div>
</template>

<script>
    export default {
        name: "module2",
        data() {
            return {
                loading: false,
                valid: false,
                inputs: [''],
                lastUsedForm: null,
                lastUsedInputIndex: 0,
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
                formula: '',
                answer: null,
                headers: [],
                items: [],

            }
        },
        methods: {
            addInput() {
                this.inputs.push('');
                this.inputs = this.inputs.map((el)=>el.toUpperCase());
                this.formula = this.formula.toUpperCase();
            },
            changeActiveForm(index) {
                if (index !== -123) {
                    this.lastUsedForm = this.$refs.forms[index];
                    this.lastUsedInputIndex = index;
                } else {
                    this.lastUsedForm = this.$refs.formula;
                    this.lastUsedInputIndex = null;
                }
            },
            insertSpecialSymbol(signIndex) {
                if (this.lastUsedInputIndex !== null) {
                    this.inputs[this.lastUsedInputIndex] += this.operations[signIndex].sign;
                    this.lastUsedForm.focus();
                } else {
                    this.formula += this.operations[signIndex].sign;
                    this.lastUsedForm.focus();
                }
                this.$forceUpdate();
            },
            async getAnswer() {
                this.loading = true;
                this.inputs = this.inputs.map((el)=>el.toUpperCase());
                this.formula = this.formula.toUpperCase();

                let inputs = this.inputs;
                let formula = this.formula;

                try {
                    let responce = await fetch(
                        'http://83.166.240.14:8080/api/consequence', {
                            method: 'POST',
                            headers: {
                                'Content-Type': 'application/json'
                            },
                            body: JSON.stringify({data: inputs, formula: formula})
                        });
                    let json = await responce.json();
                    if (!json.error) {
                        this.answer = (json.result) ? 'Верно' : 'Не верно';

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
        created() {
            setTimeout(() => {
                this.lastUsedForm = this.$refs.forms[0];
            }, 500);
        },
    }
</script>

