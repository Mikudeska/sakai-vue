<script setup>
import { FilterMatchMode } from '@primevue/core/api';
import { useToast } from 'primevue/usetoast';
import { onMounted, ref } from 'vue';
import axios from 'axios';

async function fetchPersons() {
    try {
        const response = await axios.get('http://127.0.0.1:8000/api/person/')
        persons.value = response.data.map(person => ({
            ...person,
            formatted_id: formatId(person.id) // ใช้ฟังก์ชันจัดรูปแบบ ID
        }))
    } catch (error) {
        console.error('Error fetching persons:', error)
    }
}   
onMounted(fetchPersons)

const toast = useToast();
const dt = ref();
// const persons = ref();
const persons = ref();
const productDialog = ref(false);
const deleteProductDialog = ref(false);
const deletepersonsDialog = ref(false);
const product = ref({});
const selectedpersons = ref();
const filters = ref({
    global: { value: null, matchMode: FilterMatchMode.CONTAINS }
});
const submitted = ref(false);

function formatId(id) {
    return id.toString().padStart(4, '0')
}

function openNew() {
    product.value = {};
    submitted.value = false;
    productDialog.value = true;
}

function hideDialog() {
    productDialog.value = false;
    submitted.value = false;
}

function saveProduct() {
    submitted.value = true;

    if (product?.value.name?.trim()) {
        if (product.value.id) {
            persons.value[findIndexById(product.value.id)] = product.value;
            toast.add({ severity: 'success', summary: 'บันทึกสำเร็จ', detail: 'อัพเดตข้อมูลเรียบร้อย', life: 3000 });
        } else {
            persons.value.push(product.value);
            toast.add({ severity: 'success', summary: 'บันทึกสำเร็จ', detail: 'Product Created', life: 3000 });
        }

        productDialog.value = false;
        product.value = {};
    }
}

function editProduct(prod) {
    product.value = { ...prod };
    productDialog.value = true;
}

function confirmDeleteProduct(prod) {
    product.value = prod;
    deleteProductDialog.value = true;
}

function deleteProduct() {
    persons.value = persons.value.filter((val) => val.id !== product.value.id);
    deleteProductDialog.value = false;
    product.value = {};
    toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Deleted', life: 3000 });
}

function findIndexById(id) {
    let index = -1;
    for (let i = 0; i < persons.value.length; i++) {
        if (persons.value[i].id === id) {
            index = i;
            break;
        }
    }

    return index;
}

function exportCSV() {
    dt.value.exportCSV();
}

function confirmDeleteSelected() {
    deletepersonsDialog.value = true;
}

function deleteSelectedpersons() {
    persons.value = persons.value.filter((val) => !selectedpersons.value.includes(val));
    deletepersonsDialog.value = false;
    selectedpersons.value = null;
    toast.add({ severity: 'success', summary: 'Successful', detail: 'persons Deleted', life: 3000 });
}
</script>

<template>
    <div>
        <div class="card">
            <Toolbar class="mb-6">
                <template #start>
                    <Button label="New" icon="pi pi-plus" severity="secondary" class="mr-2" @click="openNew" />
                    <Button label="Delete" icon="pi pi-trash" severity="secondary" @click="confirmDeleteSelected" :disabled="!selectedpersons || !selectedpersons.length" />
                </template>

                <template #end>
                    <Button label="Export" icon="pi pi-upload" severity="secondary" @click="exportCSV($event)" />
                </template>
            </Toolbar>

            <DataTable
                ref="dt"
                v-model:selection="selectedpersons"
                :value="persons"
                dataKey="id"
                :paginator="true"
                :rows="10"
                :filters="filters"
                paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                :rowsPerPageOptions="[5, 10, 25]"
                currentPageReportTemplate="Showing {first} to {last} of {totalRecords} persons"
                :sortField="'formatted_id'"
                :sortOrder="1"
            >
                <template #header>
                    <div class="flex flex-wrap gap-2 items-center justify-between">
                        <h4 class="m-0">Manage persons</h4>
                        <IconField>
                            <InputIcon>
                                <i class="pi pi-search" />
                            </InputIcon>
                            <InputText v-model="filters['global'].value" placeholder="ค้นหาข้อมูลบัญฑิต" />
                        </IconField>
                    </div>
                </template>

                <Column selectionMode="multiple" style="width: 3rem" :exportable="false"></Column>
                <Column field="formatted_id" header="ลำดับที่" sortable style="min-width: 6rem"></Column>
                <Column field="nisit" header="รหัสนิสิต" sortable style="min-width: 10rem"></Column>
                <!-- <Column header="Image">
                    <template #body="slotProps">
                        <img :src="`https://primefaces.org/cdn/primevue/images/product/${slotProps.data.image}`" :alt="slotProps.data.image" class="rounded" style="width: 64px" />
                    </template>
                </Column> -->
                <Column field="name" header="ชื่อ-นามสกุล" sortable style="min-width: 12rem"></Column>
                <Column field="degree" header="ชื่อปริญญา" sortable style="min-width: 12rem"></Column>
                <!-- <Column field="rating" header="Reviews" sortable style="min-width: 12rem">
                    <template #body="slotProps">
                        <Rating :modelValue="slotProps.data.rating" :readonly="true" />
                    </template>
                </Column> -->
                <Column field="seat" header="เลขที่นั่ง" sortable style="min-width: 6rem"></Column>
                <Column field="verified" header="รายงานตัว" dataType="boolean" bodyClass="text-center" style="min-width: 8rem">
                    <template #body="{ data }">
                        <i class="pi" :class="{ 'pi-check-circle text-green-500 ': data.verified, 'pi-times-circle text-red-500': !data.verified }"></i>
                    </template>
                    <template #filter="{ filterModel }">
                        <label for="verified-filter" class="font-bold"> Verified </label>
                        <Checkbox v-model="filterModel.value" :indeterminate="filterModel.value === null" binary inputId="verified-filter" />
                    </template>
                </Column>
                <Column :exportable="false" style="min-width: 1rem">
                    <template #body="slotProps">
                        <Button icon="pi pi-pencil" outlined rounded class="mr-2" @click="editProduct(slotProps.data)" />
                        <Button icon="pi pi-trash" outlined rounded severity="danger" @click="confirmDeleteProduct(slotProps.data)" />
                    </template>
                </Column>
            </DataTable>
        </div>

        <Dialog v-model:visible="productDialog" :style="{ width: '450px' }" header="Product Details" :modal="true">
            <div class="flex flex-col gap-6">
                <div>
                    <label for="nisit" class="block font-bold mb-3">รหัสนิสิต</label>
                    <InputText id="nisit" v-model.trim="product.nisit" required="true" autofocus :invalid="submitted && !product.nisit" fluid />
                    <small v-if="submitted && !product.nisit" class="text-red-500">จำเป็นต้องใส่</small>
                </div>
                <div>
                    <label for="name" class="block font-bold mb-3">ชื่อ-นามสกุล</label>
                    <InputText id="name" v-model.trim="product.name" required="true" autofocus :invalid="submitted && !product.name" fluid />
                    <small v-if="submitted && !product.name" class="text-red-500">จำเป็นต้องใส่</small>
                </div>
                <div>
                    <label for="degree" class="block font-bold mb-3">ชื่อปริญญา</label>
                    <InputText id="degree" v-model.trim="product.degree" required="true" autofocus :invalid="submitted && !product.degree" fluid />
                    <small v-if="submitted && !product.degree" class="text-red-500">จำเป็นต้องใส่</small>
                </div>
                <!-- <div>
                    <label for="description" class="block font-bold mb-3">Description</label>
                    <Textarea id="description" v-model="product.description" required="true" rows="3" cols="20" fluid />
                </div>
                <div>
                    <label for="inventoryStatus" class="block font-bold mb-3">Inventory Status</label>
                    <Select id="inventoryStatus" v-model="product.inventoryStatus" :options="statuses" optionLabel="label" placeholder="Select a Status" fluid></Select>
                </div>

                <div>
                    <span class="block font-bold mb-4">Category</span>
                    <div class="grid grid-cols-12 gap-4">
                        <div class="flex items-center gap-2 col-span-6">
                            <RadioButton id="category1" v-model="product.category" name="category" value="Accessories" />
                            <label for="category1">Accessories</label>
                        </div>
                        <div class="flex items-center gap-2 col-span-6">
                            <RadioButton id="category2" v-model="product.category" name="category" value="Clothing" />
                            <label for="category2">Clothing</label>
                        </div>
                        <div class="flex items-center gap-2 col-span-6">
                            <RadioButton id="category3" v-model="product.category" name="category" value="Electronics" />
                            <label for="category3">Electronics</label>
                        </div>
                        <div class="flex items-center gap-2 col-span-6">
                            <RadioButton id="category4" v-model="product.category" name="category" value="Fitness" />
                            <label for="category4">Fitness</label>
                        </div>
                    </div>
                </div>

                <div class="grid grid-cols-12 gap-4">
                    <div class="col-span-6">
                        <label for="price" class="block font-bold mb-3">Price</label>
                        <InputNumber id="price" v-model="product.price" mode="currency" currency="USD" locale="en-US" fluid />
                    </div>
                    <div class="col-span-6">
                        <label for="quantity" class="block font-bold mb-3">Quantity</label>
                        <InputNumber id="quantity" v-model="product.quantity" integeronly fluid />
                    </div>
                </div> -->
            </div>

            <template #footer>
                <Button label="Cancel" icon="pi pi-times" text @click="hideDialog" />
                <Button label="Save" icon="pi pi-check" @click="saveProduct" />
            </template>
        </Dialog>

        <Dialog v-model:visible="deleteProductDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
            <div class="flex items-center gap-4">
                <i class="pi pi-exclamation-triangle !text-3xl" />
                <span v-if="product"
                    >Are you sure you want to delete <b>{{ product.name }}</b
                    >?</span
                >
            </div>
            <template #footer>
                <Button label="No" icon="pi pi-times" text @click="deleteProductDialog = false" />
                <Button label="Yes" icon="pi pi-check" @click="deleteProduct" />
            </template>
        </Dialog>

        <Dialog v-model:visible="deletepersonsDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
            <div class="flex items-center gap-4">
                <i class="pi pi-exclamation-triangle !text-3xl" />
                <span v-if="product">Are you sure you want to delete the selected persons?</span>
            </div>
            <template #footer>
                <Button label="No" icon="pi pi-times" text @click="deletepersonsDialog = false" />
                <Button label="Yes" icon="pi pi-check" text @click="deleteSelectedpersons" />
            </template>
        </Dialog>
    </div>
</template>
