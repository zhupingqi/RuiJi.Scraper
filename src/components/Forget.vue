﻿<template>
    <div id="forget">
        <div class="form-signin">
            <img src="img/icon128.png">
        </div>
        <form method="POST" class="form-signin">
            <div class="form-group">
                <input name="email" type="email" class="form-control" required="required" v-bind:placeholder="i18n('email')" id="email" v-model="email">
            </div>
            <div class="form-group">
                <div class="form-row justify-content-between" style="padding-left: 5px;padding-right: 5px;">
                    <input name="captcha" type="text" class="form-control col-5" required="required" v-bind:placeholder="i18n('captcha')" id="captcha" v-model="captcha">
                    <template v-if="countDown!==-1">
                        <button name="sendEmail" type="button" class="btn btn-info col-5" disabled="disabled">{{countDown+" "+i18n("seconds")}}</button>
                    </template>
                    <template v-else>
                        <button name="sendEmail" type="button" class="btn btn-info col-5" @click="sendEmail">{{i18n("sendEmail")}}</button>
                    </template>
                </div>
            </div>
            <div class="form-group">
                <input name="password" type="password" class="form-control" required="required" v-bind:placeholder="i18n('password')" id="password" v-model="password">
            </div>
            <div class="form-group">
                <input name="password2" type="password" class="form-control" required="required" v-bind:placeholder="i18n('passwordAgain')" id="password2" v-model="password2">
            </div>
            <p class="text-danger" v-if="message!==''"><span class="glyphicon glyphicon-warning-sign" aria-hidden="true"></span>{{ message }}</p>
            <button name="submit" type="submit" class="btn  btn-info btn-block" id="submit">{{i18n("commit")}}</button>
        </form>
        <p class="form-signin">
            <span class="float-left"><a class="" href="login.html">{{i18n("login")}}</a></span>
            <span class="float-right"><a class="" href="register.html">{{i18n("register")}}</a></span>
        </p>
    </div>
</template>

<script lang="ts">
    import $ from 'jquery';
    import { Component, Prop, Vue } from 'vue-property-decorator';
    import { _browser } from '@/core/utils';

    @Component
    export default class Register extends Vue {
        email: string = '';
        password: string = '';
        password2: string = '';
        captcha: string = '';
        message: string = '';
        countDown: number = -1;

        sendEmail() {
            var _this = this;

            if (this.email === "") {
                this.message = _this.i18n("please", "input", "email");
                return false;
            }

            var times = 60;
            var timer = setInterval(function () {
                if (times >= 0) {
                    _this.countDown = times--;
                } else {
                    clearInterval(timer);
                    _this.countDown = -1;
                }
            }, 1000);

            _browser.sendMessage({ cmd: "sendMail", email: this.email }, (response) => {
                if (response.code !== 200) {
                    alert(response.msg);
                    clearInterval(timer);
                    _this.countDown = -1;
                }
            });
        }

        submit() {
            var _this = this;
            this.message = "";

            if (this.password !== this.password2) {
                $("input[name='password']").focus();
                this.password = this.password2 = "";
                this.message = _this.i18n("pwd_twice");

                return;
            }

            if (this.captcha.length === 0) {
                this.message = _this.i18n("please", "input", "captcha");

                return;
            }

            var d = {
                email: this.email,
                password: this.password
            };

            _browser.sendMessage({ cmd: "user.forget", data: d, captcha: this.captcha }, (response) => {
                if (response.code === 401) {
                    _this.message = _this.i18n("captcha", "error");
                }
                if (response.code === 402) {
                    _this.message = _this.i18n("user_exists");
                }
                if (response.code === 200) {
                    alert(_this.i18n("forget_update_success"));
                    window.location.replace("settings.html");
                }
                if (response.code === -1) {
                    _this.message = _this.i18n("error_unknown");
                }
            });
        }

        mounted() {
            let _this = this;

            $('form').submit(() => {
                _this.submit();

                return false;
            });
        }
    }
</script>

<style>
    body {
        padding-top: 40px;
        padding-bottom: 40px;
        background-color: #eee;
        font-size: 14px;
    }

    .form-signin {
        max-width: 330px;
        padding: 15px;
        margin: 0 auto;
        text-align: center;
    }

        .form-signin .form-signin-heading,
        .form-signin .checkbox {
            margin-bottom: 10px;
        }

        .form-signin .checkbox {
            font-weight: normal;
        }

        .form-signin .form-control {
            position: relative;
            height: auto;
            -webkit-box-sizing: border-box;
            -moz-box-sizing: border-box;
            box-sizing: border-box;
            padding: 10px;
            font-size: 16px;
        }

            .form-signin .form-control:focus {
                z-index: 2;
            }

    .input-120 {
        width: 120px;
        margin-bottom: 15px;
    }

    .form-signin input[type="email"] {
        margin-bottom: -1px;
        border-bottom-right-radius: 0;
        border-bottom-left-radius: 0;
    }

    .form-signin input[type="password"] {
        margin-bottom: 10px;
        border-top-left-radius: 0;
        border-top-right-radius: 0;
    }
</style>