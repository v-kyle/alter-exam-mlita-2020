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
                    <template v-for="(atom, index) in atoms">
                        <v-col :key="index">
                            <v-btn @click="insertAtom(index)">{{ atom.str }}</v-btn>
                        </v-col>
                    </template>
                </v-row>
                <v-row>
                    <v-col>
                        <v-btn color="primary white--text" @click="getAnswer" :disabled="!valid">
                            Проверить
                        </v-btn>
                    </v-col>
                    <v-col>
                        <v-btn color="primary white--text" @click="onlyConvert">
                            Преобразовать посылки
                        </v-btn>
                    </v-col>
                </v-row>
            </v-container>
        </v-form>
        <v-progress-linear :active="loading" :indeterminate="loading"/>
        <v-alert color="success" dense class="white--text">
            Результат: <br/> <b>{{ answer }}</b>
        </v-alert>

        <v-list v-if="!error">
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
    /* eslint-disable no-console */

    export default {
        name: "sentences",
        data() {
            return {
                error: false,
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
                atoms: [],
                formula: '',
                answer: null,
                lastUsedCharCode: 65,
            }
        },
        methods: {
            convertToWords(value) {
                let str1 = null;
                let str2 = null;
                let atom1 = null;
                let atom2 = null;

                let type = null;
                if (value.match(/\sИ\s/gi)) type = '*';
                if (value.match(/\sИли\s/gi)) type = '+';
                if (value.match(/^Если\s/gi) && value.match(/\sто\s/gi)) {
                    type = '->';
                }
                if (value.match(/тогда и только тогда/gi)) type = '<=>';

                switch (type) {
                    case '->':
                        str1 = value.slice(value.match(/^Если\s/i).index + 5, value.match(/,/i).index);
                        str2 = value.slice(value.match(/\sто\s/i).index + 4, value.length);

                        if (!this.atoms.map(el => el.str).includes(str1)) {
                            atom1 = {letter: String.fromCharCode(this.lastUsedCharCode++), str: str1};
                            this.atoms.push(atom1);
                        } else {
                            let index = this.atoms.map(el => el.str).indexOf(str1);
                            atom1 = {letter: this.atoms[index].letter, str: str1};
                        }

                        if (!this.atoms.map(el => el.str).includes(str2)) {
                            atom2 = {letter: String.fromCharCode(this.lastUsedCharCode++), str: str2};
                            this.atoms.push(atom2);
                        } else {
                            let index = this.atoms.map(el => el.str).indexOf(str2);
                            atom2 = {letter: this.atoms[index].letter, str: str2};
                        }

                        return `${atom1.letter} → ${atom2.letter}`;
                    case '*':

                        break;
                }

                return value;
            },

            addInput() {
                this.inputs.push('');

                // this.inputs = this.inputs.map(this.convertSymbols);
                // this.formula = this.convertSymbols(this.formula);

                this.lastUsedForm = this.$refs.forms[this.$refs.forms.length - 1];
                this.lastUsedInputIndex = this.inputs.length - 1;

                setTimeout(() => {
                    this.$refs.forms[this.$refs.forms.length - 1].focus();
                }, 100);

            },
            removeLastInput() {
                this.inputs.pop();
                this.$refs.forms.pop();

                // this.inputs = this.inputs.map(this.convertSymbols);
                // this.formula = this.convertSymbols(this.formula);

                this.lastUsedForm = this.$refs.forms[this.$refs.forms.length - 1];
                this.lastUsedInputIndex = this.inputs.length - 1;

                setTimeout(() => {
                    this.$refs.forms[this.$refs.forms.length - 1].focus();
                }, 100);
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
            insertAtom(atomIndex) {
                if (this.lastUsedInputIndex !== null) {
                    this.inputs[this.lastUsedInputIndex] += this.atoms[atomIndex].str;
                } else {
                    this.formula += this.atoms[atomIndex].str;
                    this.lastUsedForm.focus();
                }
                this.$forceUpdate();
            },

            async onlyConvert() {
                this.inputs = await this.inputs.map(this.convertToWords);
                this.formula = await this.convertToWords(this.formula);
            },

            async getAnswer() {
                this.error = false;
                this.loading = true;

                this.inputs = await this.inputs.map(this.convertToWords);
                this.formula = await this.convertToWords(this.formula);

                this.inputs = await this.inputs.map(input=>{
                    if (this.atoms.map(el=>el.str).includes(input)) {
                        input = this.atoms[this.atoms.map(el=>el.str).indexOf(input)].letter;
                    }
                    return input;
                });

                if (this.atoms.map(el=>el.str).includes(this.formula)) {
                    this.formula = this.atoms[this.atoms.map(el=>el.str).indexOf(formula)].letter;
                }

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
                    } else {
                        this.answer = "Ошибка ввода!";
                        this.error = true;
                    }
                } catch (e) {
                    this.error = true;
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
