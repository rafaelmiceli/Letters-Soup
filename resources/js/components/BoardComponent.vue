<template>
    <div class="container-fluid">
        <div class="row justify-content-center">
            <div class="col-md-12">
                <div class="card card-default">
                    <div class="card-header">
                        <span>Search Pattern in an Alphabet Soups</span>
                        <b-button @click="generate_matrix" class="float-right" variant="success">Generate Matrix</b-button>
                    </div>
                    <div class="card-body">
                        <div class="col-md-12 mb-2" v-if="matrix.length>0">
                            <b-form inline>
                                <label class="mr-sm-2" for="inline-form-custom-select-pref">Pattern {{pattern}}:</label>
                                <label class="mr-sm-2" for="inline-form-custom-select-pref">Matrix:</label>
                                <b-form-select
                                    id="inline-form-custom-select-pref"
                                    class="mb-2 mr-sm-2 mb-sm-0"
                                    :options="options"
                                    :value="null"
                                    v-model="midx"
                                    ></b-form-select>
                                <b-button  
                                    type="button" 
                                    @click="check()" 
                                    variant="primary" 
                                    class="mr-4">Find</b-button>
                            </b-form>
                        </div>
                        <div class="col-md-12 mb-2" v-if="results.length>0">
                            <label class="mr-sm-2" for="inline-form-custom-select-pref">El Pattern {{pattern}} appears:</label>
                            <ul>
                                <li v-for="(result, index) in results" :key="index">{{result}}</li>
                            </ul>
                        </div>
                        <matrix-component 
                            :matrix="m" 
                            :index="idx" 
                            v-for="(m,idx) in matrix" 
                            :key="idx"
                            v-on:remove-matrix="onRemoveMatrix"
                            ></matrix-component>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from "axios";

export default {
    data() {
        return {
            count: 0,
            matrix: [],
            cols: 0,
            rows: 0,
            max_cols: 8,
            max_rows: 8,
            min_cols: 3,
            min_rows: 3,
            results: [],
            midx: null,
            pattern: 'OIE',
            options: []
        };
    },
    watch: {
        matrix: function (matrix) {
            this.options = []
            this.options.push({
                text: 'All Matrixs',
                value: -1
            })
            for (let index = 0; index < matrix.length; index++) {
                this.options.push({
                    text: 'Matrix ' + (index + 1),
                    value: index
                })
            }
            this.midx = -1;
        },
    },
    methods: {
        generate_matrix() {
            let rows = [], index = this.matrix.length, 
                nrows = this.generate_random(this.min_rows, this.max_rows+1),
                ncols = this.generate_random(this.min_cols, this.max_cols+1)

            for (let i = 1; i < nrows; i++) {
                let cols = []
                for (let j = 1; j < ncols; j++) {
                    cols.push(this.generate_letter(true));
                }
                rows.push(cols);
                cols.slice()
            }
            this.matrix.push(rows);
        },
        onRemoveMatrix(index) {
            this.matrix.splice(index, 1)
            this.results = []
        },
        async check() {
            if(this.midx==null) {
                return false;
            }
            this.results = []
            let matrixes = (this.midx == -1)
                ? this.matrix
                : [this.matrix[this.midx]]
            try {
                const response = await axios.post("/api/check", {
                        pattern: this.pattern,
                        matrixes: matrixes
                    }, {
                    headers: {
                        Accept: "application/json",
                        "Content-Type": "application/json"
                    }
                }).then(response => {
                    this.results = response.data.results;
                });
            } catch (error) {
                console.log(error);
            }
        },
        generate_letter(forceVocals = false) {
            let max = null, min = null, aux = [], random = null, charCode = null
            if(forceVocals) {
                aux = [79,73,69,85,65] //force vocals
                max = aux.length
                random = this.generate_random(min,max);
                charCode = aux[random];
            } else {
                min = 90;
                max = 65;
                charCode = this.generate_random(min,max);
            }
            return String.fromCharCode(charCode);
        },
        generate_random(min,max) {
            return Math.floor(Math.random() * (max - min) ) + min;
        }
    }
}
</script>