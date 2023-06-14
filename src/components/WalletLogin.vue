<template>
    <div>
        <p>
            瀏覽器名稱： {{ browserName }} <br>
            瀏覽器版本： {{ browserVersion }} <br>
            作業系統： {{ os }} <br>
            平台： {{ platform }} <br>
            渲染引擎： {{ engine }} <br>
        </p>
        
        <template v-if="isMetaMaskInstalled">
            <h1>連接常用錢包</h1>
            <button @click="connectMetamask">使用 Metamask 登入</button>
            <div v-if="loggedIn">
                <p>已登入，帳戶地址: {{ account }}</p>
            </div>
            <div v-else>
                <p>請使用支援Web3.0的錢包進行登入</p>
            </div>

            <button @click="logout('Metamask')">登出Metamask錢包</button>
            {{ msg }}
        </template>
        <template v-else>
            MetaMask 未安裝，請點選<a :href="extendLink()" target="_blank">擴充套件</a>
        </template>       
        
    </div>
</template>
<script>
import Web3 from 'web3';
import Bowser from "bowser";

const bowserType = {
    chrome: { name: 'Chrome', desktopLink: 'https://chrome.google.com/webstore/detail/metamask/nkbihfbeogaeaoehlefnkodbefgpgknn?hl=zh-TW', mobileLink: '' },
    firefox: { name: 'Firefox', desktopLink: 'https://addons.mozilla.org/zh-TW/firefox/addon/ether-metamask/', mobileLink: '' },
    edge: { name: 'Microsoft Edge', desktopLink: 'https://microsoftedge.microsoft.com/addons/detail/metamask/ejbalbakoplchlghecdalmeeeajnimhm', mobileLink: '' },
    safari: { name: 'Safari', desktopLink: '', mobileLink: '' },
}

export default {
    data() {
        return {
            bowserType: bowserType,
            browserName: null,
            browserVersion: null,
            os: null,
            platform: null,
            engine: null,

            web3: null,
            loggedIn: false,
            account: null,
            msg: null,
        };
    },
    computed: {
        isMetaMaskInstalled() {
            // 檢查 MetaMask 對象是否存在於全局作用域中
            return typeof window.ethereum !== 'undefined';
        },
        selected() {
            return Object.values(bowserType).filter(e => e.name == this.browserName)[0];
        }
    },
    methods: {
        extendLink() {
            const selectedBowserType = Object.values(bowserType).filter(e => e.name == this.browserName)[0];
            if(this.platform == 'desktop') return selectedBowserType.desktopLink;
            else if (this.platform == 'mobile') return selectedBowserType.mobileLink;
        },
        async connectMetamask() {
            if (window.ethereum) {
                this.web3 = new Web3(window.ethereum);
                try {
                    // 請求使用者授權
                    await window.ethereum.enable();
                    // 獲取帳戶
                    const accounts = await this.web3.eth.getAccounts();
                    this.account = accounts[0];
                    this.loggedIn = true;
                    // 在這裡可以執行其他Web3相關的操作
                } catch (error) {
                    console.error(error);
                }
            } else {
                console.log('請安裝MetaMask錢包插件');
            }
        },
        logout() {
            if (this.isMetaMaskInstalled) {
                window.ethereum.request({ method: 'wallet_requestPermissions', params: [{ eth_accounts: {} }] })
                .then(() => {
                    this.msg = '已登出錢包';
                    this.account = null;
                    this.loggedIn = false;
                    setTimeout(() => {this.msg = ''}, 3000)
                })
                .catch((error) => {
                    this.msg = '無法登出：' + error;
                });
            }
        }
    },
    created() {
        const browser = Bowser.getParser(window.navigator.userAgent);
        this.browserName = browser.getBrowserName();
        this.browserVersion = browser.getBrowserVersion();
        this.os = browser.getOSName();
        this.platform = browser.getPlatformType();
        this.engine = browser.getEngineName();
    }
};
</script>