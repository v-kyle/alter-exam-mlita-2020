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
                <v-row>
                    <v-col>
                        <v-btn @click="addInput">Добавить поле</v-btn>
                    </v-col>
                    <v-col>
                        <v-btn @click="removeLastInput" :disabled="inputs.length === 1">Удалить последнее поле</v-btn>
                    </v-col>
                </v-row>
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
                    Проверить
                </v-btn>
            </v-container>
        </v-form>
        <v-progress-linear :active="loading" :indeterminate="loading"/>
        <v-alert color="success" dense class="white--text">
            Результат: <br/> <b>{{ answer }}</b>
        </v-alert>

        <v-list>
            <v-list-item-group v-model="model">
                <v-list-item
                        v-for="(item, i) in items"
                        :key="i"
                >
                    <v-list-item-content>
                        <v-list-item-title v-text="item"/>
                    </v-list-item-content>
                </v-list-item>
            </v-list-item-group>
        </v-list>
    </div>
</template>

<script>
    export default {
        name: "module2",
        data() {
            return {
                model: 1,
                items: [],
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

            addInput() {
                this.inputs.push('');
                this.inputs = this.inputs.map(this.convertSymbols);
                this.formula = this.convertSymbols(this.formula);
            },
            removeLastInput() {
                this.inputs.pop();
                this.$refs.forms.pop();

                this.inputs = this.inputs.map(this.convertSymbols);
                this.formula = this.convertSymbols(this.formula);

                this.lastUsedForm = this.$refs.forms[this.$refs.forms.length - 1];
                this.lastUsedInputIndex = this.inputs.length - 1;
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
                this.inputs = this.inputs.map(this.convertSymbols);
                this.formula = this.convertSymbols(this.formula);

                let inputs = this.inputs;
                let formula = this.formula;

                try {
                    let responce = await fetch(
                        'http://83.166.240.14:8080/api/resolution', {
                            method: 'POST',
                            headers: {
                                'Content-Type': 'application/json'
                            },
                            body: JSON.stringify({data: inputs, formula: formula})
                        });
                    let json = await responce.json();
                    if (!json.error) {
                        this.answer = (json.result) ? 'Верно' : 'Не верно';

                        this.items = json.description;
                    }
                    else {
                        this.answer = "Ошибка ввода!";
                    }
                } catch (e) {
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

