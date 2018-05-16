<template>
    <v-app>
        <v-toolbar>

            <v-toolbar-title>VueJs Teste</v-toolbar-title>
            <v-spacer></v-spacer>

        </v-toolbar>
        <v-content>
            <v-container>
                <v-dialog v-model="dialog" max-width="500px">
                    <v-btn slot="activator" color="primary" dark class="mb-2">Nova Conta</v-btn>
                    <v-card>
                        <v-card-title>
                            <span class="headline">{{ formTitle }}</span>
                        </v-card-title>
                        <v-card-text>
                            <v-container grid-list-md>

                                <v-form wrap ref="form" v-model="validate" lazy-validation>
                                    <v-layout wrap>
                                        <v-flex xs12 sm6 md6>
                                            <v-text-field prefix="R$" type="number" v-model="editConta.value"
                                                          :rules="regras.valor" label="Valor"></v-text-field>
                                        </v-flex>
                                        <v-flex xs12 sm6 md6>
                                            <v-select
                                                    :items="[{ text: 'A pagar' },{ text: 'A receber' }]"
                                                    v-model="editConta.type"
                                                    required
                                                    :rules="regras.tipo"
                                                    label="Tipo"
                                                    class="input-group"
                                                    item-value="text"
                                            ></v-select>
                                        </v-flex>
                                        <v-flex xs12 sm6 md6>
                                            <v-select
                                                    :rules="regras.categoria"
                                                    required
                                                    :items="[{ text: 'Impostos' },{ text: 'Pessoal' }, {text: 'Produtos'}, {text: 'Serviços'}, {text: 'Outros'}]"
                                                    v-model="editConta.category"
                                                    label="Categoria"
                                                    class="input-group"
                                                    item-value="text"
                                            ></v-select>
                                        </v-flex>
                                        <v-flex xs12 sm6 md6>
                                            <v-menu
                                                    ref="menu1"
                                                    :close-on-content-click="false"
                                                    v-model="menu1"
                                                    :nudge-right="40"
                                                    lazy
                                                    transition="scale-transition"
                                                    offset-y
                                                    full-width
                                                    max-width="290px"
                                                    min-width="290px"
                                            >
                                                <v-text-field
                                                        slot="activator"
                                                        v-model="dateFormatted"
                                                        label="Date"
                                                        :rules="regras.data"
                                                        persistent-hint
                                                        @blur="date = parseDate(dateFormatted)"
                                                ></v-text-field>
                                                <v-date-picker locale="pt-br" v-model="date" no-title
                                                               @input="menu1 = false"></v-date-picker>
                                            </v-menu>
                                        </v-flex>
                                    </v-layout>

                                </v-form>
                            </v-container>
                        </v-card-text>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="blue darken-1" flat @click.native="close(true)">Cancelar</v-btn>
                            <v-btn color="blue darken-1" flat @click.native="salvarConta">Salvar</v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-layout row wrap justify-center>
                    <v-flex xs10 md6>
                        <v-alert
                                :value="mensagemSucesso"
                                type="success"
                                transition="scale-transition"
                        >
                            {{mensagemSucesso}}
                        </v-alert>
                    </v-flex>

                </v-layout>
                <v-data-table
                        :loading="loading"
                        :headers="headers"
                        :items="contas"
                        hide-actions
                        class="elevation-1"
                >
                    <template slot="items" slot-scope="props" v-if="props.item.category">
                        <td class="text-xs-left">{{ props.item.category }}</td>
                        <td class="text-xs-left">{{ formatDate(props.item.maturityDate) }}</td>
                        <td class="text-xs-left">{{ props.item.type == 'receivable' ? 'A receber': 'A pagar' }}</td>
                        <td class="text-xs-left">{{ props.item.value }}</td>
                        <td class="text-xs-left layout px-0">
                            <v-btn icon class="mx-0" @click="editarConta(props.item)">
                                <v-icon color="teal">edit</v-icon>
                            </v-btn>
                            <v-btn icon class="mx-0" @click="deletarConta(props.item)">
                                <v-icon color="pink">delete</v-icon>
                            </v-btn>
                        </td>
                    </template>

                </v-data-table>
            </v-container>

        </v-content>
    </v-app>
</template>

<script lang="js">
    import axios from 'axios';

    export default {
        data: () => ({
            validate: true,
            mensagemSucesso: null,
            loading: false,
            dialog: false,
            date: null,
            dateFormatted: null,
            menu1: false,
            regras: {
                valor: [
                    v => !!v || 'Valor é obrigatório ',
                    v => (v > 0 && v <= 881.90) || 'Valor deve ter entre 0 e 881,90'
                ],
                tipo: [
                    v => !!v || 'Tipo é obrigatório '
                ],
                categoria: [
                    v => !!v || 'Categoria é obrigatória '
                ],
                data: [
                    v => !!v || 'Data de Vencimento é obrigatória '
                ]
            },
            headers: [
                {text: 'Categoria', sortable: false, value: 'category'},
                {text: 'Data de vencimento', sortable: false, value: 'maturityDate'},
                {text: 'Tipo', sortable: false, value: 'type'},
                {text: 'Valor', sortable: false, value: 'value'},
                {text: ' ', sortable: false, value: 'name'}
            ],
            contas: [],
            editIndex: -1,
            editConta: {
                category: '',
                type: null,
                value: null,
                maturityDate: null
            },
            contaDefault: {
                category: '',
                type: null,
                value: null,
                maturityDate: null
            }
        }),

        computed: {
            formTitle() {
                return this.editIndex === -1 ? 'Nova Conta' : 'Editar Conta'
            }
        },

        watch: {
            dialog(val) {
                val || this.close()
            },
            date(val) {
                this.dateFormatted = this.formatDate(this.date)
            }
        },

        created() {
            this.getContas()
        },

        methods: {
            formatDate(date) {
                if (!date) return null

                const [year, month, day] = date.split('-')
                return `${day}/${month}/${year}`
            },
            parseDate(date) {
                console.log(date)
                if (!date) return null

                const [day, month, year] = date.split('/')
                return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`
            },
            getContas() {
                //lista todas as contas
                this.loading = true;
                axios.get('http://finance-manager.renantaranto.com:8080/standard-accounts')
                    .then((response) => {
                        this.loading = false;
                        this.contas = response.data;
                    })
                    .catch((error) => {
                        console.log(error);
                    });
            },
            editarConta(item) {
                this.editIndex = this.contas.indexOf(item)
                this.editConta = Object.assign({}, item)
                this.editConta.type = this.editConta.type == 'receivable' ? 'A receber' : 'A pagar'
                this.date = this.editConta.maturityDate;
                this.dialog = true
            },

            deletarConta(item) {
                const index = this.contas.indexOf(item)
                //deletar a conta se o usuario clicar OK
                let result = confirm('Tem ceterza que deseja apagar esta conta?');
                if (result == true) {
                    this.loading = true;
                    axios.delete('http://finance-manager.renantaranto.com:8080/standard-accounts/' + item.id).then((response) => {
                        this.contas.splice(index, 1)
                        this.loading = false;
                        this.editConta = Object.assign({}, this.contaDefault)
                        this.editIndex = -1
                        this.mensagemSucesso = "Conta Deletada com sucesso."
                        this.$refs.form.reset();
                        setTimeout(() => {
                            this.mensagemSucesso = null;
                        }, 2000)
                    }).catch((error) => {
                        console.log(error);
                    });
                }

            },

            close(cancel = false) {
                this.dialog = false;
                if (cancel) {
                    this.editConta = Object.assign({}, this.contaDefault)
                    this.editIndex = -1;
                    this.date = null;
                    this.$refs.form.reset()
                }
            },

            salvarConta() {
                if (this.editIndex > -1) {
                    //atualiza a conta se o form for validado
                    if (this.$refs.form.validate()) {
                        this.close();
                        this.loading = true;
                        this.editConta.type = this.editConta.type == "A pagar" ? "payable" : "receivable";
                        this.editConta.maturityDate = this.date;
                        axios.put('http://finance-manager.renantaranto.com:8080/standard-accounts/' + this.editConta.id, this.editConta).then((response) => {
                            this.contas[this.editIndex].category = response.data.category;
                            this.contas[this.editIndex].type = response.data.type;
                            this.contas[this.editIndex].value = response.data.value;
                            this.contas[this.editIndex].maturityDate = response.data.maturityDate;
                            this.editConta = Object.assign({}, this.contaDefault)
                            this.editIndex = -1;
                            this.date = null;
                            this.loading = false;
                            this.mensagemSucesso = "Conta Atualizada com sucesso."
                            this.$refs.form.reset();
                            setTimeout(() => {
                                this.mensagemSucesso = null;
                            }, 2000)
                        }).catch((error) => {
                            console.log(error);
                        });
                    }
                } else {
                    //insere a conta se o form for validado

                    if (this.$refs.form.validate()) {
                        this.close();
                        this.loading = true;
                        axios.post('http://finance-manager.renantaranto.com:8080/standard-accounts', {
                            "value": this.editConta.value,
                            "type": this.editConta.type == "A pagar" ? "payable" : "receivable",
                            "category": this.editConta.category,
                            "maturityDate": this.date
                        }).then((response) => {
                            this.contas.push(response.data);
                            this.editConta = Object.assign({}, this.contaDefault)
                            this.editIndex = -1;
                            this.date = null;
                            this.loading = false;
                            this.mensagemSucesso = "Conta Inserida com sucesso."
                            this.$refs.form.reset();
                            setTimeout(() => {
                                this.mensagemSucesso = null;
                            }, 2000)
                        }).catch((error) => {
                            console.log(error);
                        });
                    }

                }
            }
        }
    }
</script>
