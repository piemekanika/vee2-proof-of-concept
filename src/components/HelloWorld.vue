<template>
    <validation-observer ref="form">
        <div v-if="isLoading">
            Loading...        
        </div>
        
        <div>
            <validation-provider :rules="{ asyncCheck: model }" v-slot="{ errors }">
                <input 
                    v-model="model.userInput"
                    :disabled="isLoading"
                    placeholder="Введите число"
                >
                
                <button @click="submit">
                    готово
                </button>
                
                <div v-if="errors.length">
                    {{ errors }}
                </div>
            </validation-provider>        
        </div>        
    </validation-observer>
</template>

<script>
import { Validator, ValidationObserver, ValidationProvider } from 'vee-validate';
import Vue from 'vue';

Vue.component('ValidationProvider', ValidationProvider);
Vue.component('ValidationObserver', ValidationObserver);

const fakeApiCall = model => new Promise((res, rej) => {
    setTimeout(() => {
        const userInput = Number(model.userInput);
        
        if (userInput < model.bar && userInput > model.foo) {
            res();
        } else {
            rej(new Error('Uga buga'));
        }
    }, 1000);
})

Validator.extend('asyncCheck', {
    validate: async (val, { model }) => {
        try {
            await fakeApiCall(model);
           
            return true;
        } catch (e) {
            return false;
        }
    },
    getMessage() {
        return 'Вы ввели неверное число';
    },
}, { paramNames: ['model'] });

export default {
    name: 'HelloWorld',
    data() {
        return {
            model: {
                foo: 10,
                bar: 100,
                userInput: '',
            },
            isLoading: false,
        };
    },
    methods: {
        async submit() {
            
            this.isLoading = true;
            
            const success = await this.$refs.form.validate();
            
            if (success) {
                alert('Вы прошли валидацию!');
            }
            
            this.isLoading = false;
        },
    },
};
</script>