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
                        @change="changeActiveForm(index)"
                        :rules="rules"
                        label="Formula"
                        required
                />
                <v-btn @click="addInput">Добавить поле</v-btn>
                <v-divider class="my-4"/>
                <div><b>Формула для проверки:</b></div>
                <v-text-field
                        ref="formula"
                        v-model="formula"
                        @change="changeActiveForm(-123)"
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
                <v-btn text color="deep-purple accent-4" @click="getAnswer" :disabled="!valid">
                    Enter
                </v-btn>
            </v-container>
        </v-form>
        <v-alert color="primary" dense class="white--text">
            Result is: <b>{{ answer }}</b>
        </v-alert>
    </div>
</template>

<script>
    export default {
        name: "module2",
        data() {
            return {
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
                    {operation: 'XOR', sign: '⊕'},
                ],
                formula: '',
                answer: null,

            }
        },
        methods: {
            addInput() {
                this.inputs.push('');
            },
            changeActiveForm(index) {
                if (index !== -123) {
                    this.inputs[index] = this.inputs[index].toUpperCase();
                    this.lastUsedForm = this.$refs.forms[index];
                    this.lastUsedInputIndex = index;
                } else {
                    this.formula = this.formula.toUpperCase();
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
                let inputs = this.inputs;
                let formula = this.formula;

                try {
                    let responce = await fetch(
                        'https://cors-anywhere.herokuapp.com/http://83.166.240.14:8080/api/consequence', {
                            method: 'POST',
                            headers: {
                                'Content-Type': 'application/json'
                            },
                            body: JSON.stringify({data: inputs, formula: formula})
                        });
                    let json = await responce.json();
                    if (!json.error) {
                        this.answer = (json.result) ? 'Верно' : 'Не верно';
                    }
                } catch (e) {
                    this.answer = "Error!!!";
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

