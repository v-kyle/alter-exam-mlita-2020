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
                <v-btn text color="deep-purple accent-4" @click="getAnswer" :disabled="!valid">
                    Enter
                </v-btn>
            </v-container>
        </v-form>
        <v-divider class="mx-4"/>
        <v-alert color="primary" dense class="white--text">
            Result is: <b>{{ answer }}</b>
        </v-alert>
    </div>
</template>

<script>
    export default {
        name: "module1",
        data() {
            return {
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
                    {operation: 'XOR', sign: '⊕'},
                ],
                valid: false,
            }
        },
        methods: {
            insertSpecialSymbol(signIndex) {
                this.input += this.operations[signIndex].sign;
                this.$refs.formInput.focus();
            },

            async getAnswer() {
                let input = this.input;

                try {
                    let responce = await fetch(
                        'https://cors-anywhere.herokuapp.com/http://83.166.240.14:8080/api/inconsistency', {
                            method: 'POST',
                            headers: {
                                'Content-Type': 'application/json'
                            },
                            body: JSON.stringify({data: input})
                        });
                    let json = await responce.json();
                    if (!json.error) {
                        this.answer = (json.result)? 'Противоречива' : 'Не противоречива';
                    }
                } catch (e) {
                    this.answer = "Error!!!";
                }
            }
        },
        watch: {
            input(value) {
                this.input = value.toUpperCase();
            },
        }
    }
</script>

