<script setup>
import { ref } from "vue";

const props = defineProps(['customizations', 'socket']);

const customizations = ref(props.customizations);

const socket  = ref(props.socket);

let username = "John Doe";
let user_mail = "example@outlook.com";

const placeOrder = async () => {
    try {
        let order = {
            action: "newOrder",
            laces_color: customizations.value.laces.color,
            inside_color: customizations.value.inside.color,
            outside_1_color: customizations.value.outside_1.color,
            outside_2_color: customizations.value.outside_2.color,
            outside_3_color: customizations.value.outside_3.color,
            sole_bottom_color: customizations.value.sole_bottom.color,
            sole_top_color: customizations.value.sole_top.color,
            laces_material: "leather",
            sole_bottom_material: "leather",
            sole_top_material: "rubber",
            inside_material: "rubber",
            outside_1_material: "leather",
            outside_2_material: "rubber",
            outside_3_material: "leather",
            username: username,
            user_mail: user_mail,
        };

        socket.value.send(JSON.stringify(order));

        const response = await fetch("http://localhost:3000/api/v1/shoes/", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify(order),
        });

        if (response.ok) {
            console.log("Order placed successfully!");
        } else {
            console.error("Error placing order:", response);
        }
    } catch (error) {
        console.error("Error placing order:", error);
    }
};
</script>

<template>
    <div class="flex justify-center mt-7">
        <a @click="placeOrder" href="#" class="bg-[#69FF47] text-white px-[75px] py-2 rounded font-bold">Order</a>
    </div>
</template>