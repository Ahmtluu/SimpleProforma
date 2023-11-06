<script setup>
import axios from "axios";
import { onMounted, ref } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();

let form = ref([]);
let allCustomers = ref([]);
let customer_id = ref([]);
let item = ref([]);
let listCart = ref([]);
let showModal = ref(false);
let allProducts = ref([]);

const tax_rates = [1, 10, 20];

onMounted(async () => {
    indexForm();
    getAllCustomers();
    getAllProducts();
});

const indexForm = async () => {
    let response = await axios.get("/api/create_invoice");
    form.value = response.data;
};

const getAllCustomers = async () => {
    let response = await axios.get("/api/customers");
    allCustomers.value = response.data.customers;
};

const getAllProducts = async () => {
    let response = await axios.get("/api/products");
    allProducts.value = response.data.products;
};

const addCart = (item) => {
    const itemcart = {
        id: item.id,
        item_code: item.item_code,
        description: item.description,
        unit_price: item.unit_price,
        quantity: item.quantity,
    };

    listCart.value.push(itemcart);
    hideModal();
};

const openModal = () => {
    showModal.value = !showModal.value;
};

const hideModal = () => {
    showModal.value = !showModal.value;
};

const removeItem = (index) => {
    listCart.value.splice(index, 1);
};

const onSave = () => {
    if (listCart.value.length >= 1) {
        let subTotal = 0;
        subTotal = SubTotal();

        let grandTotal = 0;
        grandTotal = GrandTotal();

        const formData = new FormData();
        formData.append("invoice_item", JSON.stringify(listCart.value));
        formData.append("customer_id", customer_id.value);
        formData.append("date", form.value.date);
        formData.append("due_date", form.value.date);
        formData.append("number", form.value.number);
        formData.append("reference", form.value.reference);
        formData.append("tax", form.value.tax);
        formData.append(
            "terms_and_conditions",
            form.value.terms_and_conditions
        );
        formData.append("subtotal", subTotal);
        formData.append("total", grandTotal);

        axios.post("/api/add_invoice", formData);
        listCart.value = [];
        router.push({ path: "/" });
    }
};

const SubTotal = () => {
    let total = 0;

    listCart.value.map((product) => {
        total = total + product.quantity * product.unit_price;
    });

    return total;
};

const GrandTotal = () => {
    return (SubTotal() * form.value.tax) / 100 + SubTotal();
};
</script>

<template>
    <div class="container">
        <div class="invoices">
            <div class="card__header">
                <div>
                    <h2 class="invoice__title">Yeni Proforma</h2>
                </div>
                <div></div>
            </div>

            <div class="card__content">
                <div class="card__content--header">
                    <div>
                        <p class="my-1">Müşteri</p>
                        <select
                            name=""
                            id=""
                            class="input"
                            v-model="customer_id"
                        >
                            <option disabled>Seçiniz</option>
                            <option
                                v-for="customer in allCustomers"
                                :value="customer.id"
                            >
                                {{ customer.firstname }}
                            </option>
                        </select>
                    </div>
                    <div>
                        <p class="my-1">Tarih</p>
                        <input
                            id="date"
                            placeholder="dd-mm-yyyy"
                            type="date"
                            class="input"
                            v-model="form.date"
                        />
                    </div>
                    <div>
                        <p class="my-1">Numara</p>
                        <input
                            type="text"
                            class="input"
                            v-model="form.number"
                        />
                        <p class="my-1">Referans(Tercihen)</p>
                        <input
                            type="text"
                            class="input"
                            v-model="form.reference"
                        />
                    </div>
                </div>
                <br /><br />
                <div class="table">
                    <div class="table--heading2">
                        <p>Ürün</p>
                        <p>Birim Fiyatı</p>
                        <p>Adet</p>
                        <p>Toplam</p>
                        <p></p>
                    </div>

                    <!-- item 1 -->
                    <div
                        class="table--items2"
                        v-for="(itemcart, i) in listCart"
                    >
                        <p>
                            #{{ itemcart.item_code }} {{ itemcart.description }}
                        </p>
                        <p>
                            <input
                                type="number"
                                class="input"
                                v-model="itemcart.unit_price"
                            />
                        </p>
                        <p>
                            <input
                                type="number"
                                class="input"
                                v-model="itemcart.quantity"
                            />
                        </p>
                        <p v-if="itemcart.quantity">
                            $ {{ itemcart.quantity * itemcart.unit_price }}
                        </p>
                        <p v-else></p>
                        <p
                            style="color: red; font-size: 24px; cursor: pointer"
                            @click="removeItem(i)"
                        >
                            &times;
                        </p>
                    </div>
                    <div style="padding: 10px 30px !important">
                        <button
                            class="btn btn-sm btn__open--modal"
                            @click="openModal()"
                        >
                            Yeni ürün ekle
                        </button>
                    </div>
                </div>

                <div class="table__footer">
                    <div class="document-footer">
                        <p>Şartlar ve Koşullar</p>
                        <textarea
                            cols="50"
                            rows="7"
                            class="textarea"
                            v-model="form.terms_and_conditions"
                        ></textarea>
                    </div>
                    <div>
                        <div class="table__footer--subtotal">
                            <p>Ara Toplam</p>
                            <span>$ {{ SubTotal() }}</span>
                        </div>
                        <div class="table__footer--discount">
                            <p>KDV</p>

                            <select
                                name=""
                                id=""
                                class="input"
                                v-model="form.tax"
                            >
                                <option disabled>Seçiniz</option>
                                <option v-for="tax in tax_rates" :value="tax">
                                    {{ tax }}
                                </option>
                            </select>
                        </div>
                        <div class="table__footer--total">
                            <p>Genel Toplam</p>
                            <span>$ {{ GrandTotal() }}</span>
                        </div>
                    </div>
                </div>
            </div>
            <div
                class="card__header"
                style="margin-top: 40px; margin-bottom: 50px"
            >
                <div></div>
                <div>
                    <a class="btn btn-success" @click="onSave()"> Kaydet </a>
                </div>
            </div>
        </div>
        <!--==================== add modal items ====================-->
        <div class="modal main__modal" :class="{ show: showModal }">
            <div class="modal__content">
                <span class="modal__close btn__close--modal" @click="hideModal"
                    >×</span
                >
                <h3 class="modal__title">Add Item</h3>
                <hr />
                <br />
                <div class="modal__items">
                    <ul style="list-style: none">
                        <li
                            v-for="(item, i) in allProducts"
                            :key="item.id"
                            style="
                                display: grid;
                                grid-template-columns: 30px 350px 15px;
                                align-items: center;
                                padding-bottom: 5px;
                            "
                        >
                            <p>{{ i + 1 }}</p>
                            <p>{{ item.item_code }} {{ item.description }}</p>
                            <button
                                style="
                                    width: 24px;
                                    height: 24px;
                                    border: 1px solid #145236;
                                "
                                @click="addCart(item)"
                            >
                                +
                            </button>
                        </li>
                    </ul>
                </div>
                <br />
                <hr />
            </div>
        </div>
    </div>
</template>
