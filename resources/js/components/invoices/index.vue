<script setup>
import axios from "axios";
import { onMounted, ref } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();

let invoices = ref([]);
let searchInvoice = ref([]);

onMounted(async () => {
    getInvoices();
});

const getInvoices = async () => {
    let response = await axios.get("/api/get_all_invoice");
    invoices.value = response.data.invoices;
};

const search = async () => {
    let response = await axios.get(
        "/api/search_invoice?s=" + searchInvoice.value
    );
    invoices.value = response.data.invoices;
};

const newInvoice = async () => {
    let form = await axios.get("/api/create_invoice");
    router.push("/invoice/new");
};
</script>

<template>
    <div class="container">
        <div class="invoices">
            <div class="card__header">
                <div>
                    <h2 class="invoice__title">Invoices</h2>
                </div>
                <div>
                    <a @click="newInvoice" class="btn btn-secondary">
                        Yeni Proforma
                    </a>
                </div>
            </div>

            <div class="table card__content">
                <div class="table--filter">
                    <span class="table--filter--collapseBtn">
                        <i class="fas fa-ellipsis-h"></i>
                    </span>
                    <div class="table--filter--listWrapper">
                        <ul class="table--filter--list">
                            <li>
                                <p
                                    class="table--filter--link table--filter--link--active"
                                >
                                    Hepsi
                                </p>
                            </li>
                        </ul>
                    </div>
                </div>

                <div class="table--search">
                    <div class="table--search--wrapper">
                        <select class="table--search--select" name="" id="">
                            <option value="">Filtrele</option>
                        </select>
                    </div>
                    <div class="relative">
                        <i class="table--search--input--icon fas fa-search"></i>
                        <input
                            class="table--search--input"
                            type="text"
                            placeholder="Proforma Ara "
                            v-model="searchInvoice"
                            @keyup="search()"
                        />
                    </div>
                </div>

                <div class="table--heading">
                    <p>ID</p>
                    <p>Tarih</p>
                    <p>Numara</p>
                    <p>Müşteri</p>
                    <p>Toplam</p>
                </div>

                <!-- item 1 -->
                <div
                    class="table--items"
                    v-for="item in invoices"
                    v-if="invoices.length > 0"
                    :key="item.id"
                >
                    <a href="#" class="table--items--transactionId"
                        >#{{ item.id }}</a
                    >
                    <p>{{ item.date }}</p>
                    <p>{{ item.number }}</p>
                    <p v-if="item.customer">{{ item.customer.firstname }}</p>
                    <p v-else></p>
                    <p>$ {{ item.total }}</p>
                </div>

                <div class="table--items" v-else>There is no invoices</div>
            </div>
        </div>
    </div>
</template>
