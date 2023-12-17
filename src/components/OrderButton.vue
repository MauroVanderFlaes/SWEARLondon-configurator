<script setup>
import { ref } from "vue";

const props = defineProps(['customizations', 'socket']);

const customizations = ref(props.customizations);

const socket = ref(props.socket);

let username = "John Doe";
let user_mail = "example@outlook.com";

const confirmation = ref("");

const placeOrder = async () => {
    try {
        let order = {
            "laces_color": customizations.value.laces.color,
            "inside_color": customizations.value.inside.color,
            "outside_1_color": customizations.value.outside_1.color,
            "outside_2_color": customizations.value.outside_2.color,
            "outside_3_color": customizations.value.outside_3.color,
            "sole_bottom_color": customizations.value.sole_bottom.color,
            "sole_top_color": customizations.value.sole_top.color,
            "laces_material": customizations.value.laces.material,
            "inside_material": customizations.value.inside.material,
            "outside_1_material": customizations.value.outside_1.material,
            "outside_2_material": customizations.value.outside_2.material,
            "outside_3_material": customizations.value.outside_3.material,
            "sole_bottom_material": customizations.value.sole_bottom.material,
            "sole_top_material": customizations.value.sole_top.material,
            "username": username,
            "user_mail": user_mail
        };

        const response = await fetch("https://swearlondon.onrender.com/api/v1/shoes/", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify(order),
        });

        if (response.ok) {
            // Parse the response to get the order ID
            const responseData = await response.json();
            const orderId = responseData.data[0].id;
            confirmation.value = "Order is placed!"

            // Include the order ID in the WebSocket message
            const websocketMessage = {
                "action": "newOrder",
                "id": orderId,
                "laces_color": customizations.value.laces.color,
                "inside_color": customizations.value.inside.color,
                "outside_1_color": customizations.value.outside_1.color,
                "outside_2_color": customizations.value.outside_2.color,
                "outside_3_color": customizations.value.outside_3.color,
                "sole_bottom_color": customizations.value.sole_bottom.color,
                "sole_top_color": customizations.value.sole_top.color,
                "laces_material": customizations.value.laces.material,
                "inside_material": customizations.value.inside.material,
                "outside_1_material": customizations.value.outside_1.material,
                "outside_2_material": customizations.value.outside_2.material,
                "outside_3_material": customizations.value.outside_3.material,
                "sole_bottom_material": customizations.value.sole_bottom.material,
                "sole_top_material": customizations.value.sole_top.material,
                "username": username,
                "user_mail": user_mail,
                "status": "To be produced",
            };

            // Send the WebSocket message
            socket.value.send(JSON.stringify(websocketMessage));
        } else {
            console.error("Error placing order:", response);
            confirmation = "Error placing order";
        }
    } catch (error) {
        console.error("Error placing order:", error);
    }
};
</script>

<template>
    <div class="flex justify-center mt-7">
        <a @click="placeOrder" href="#"
            class="absolute top-5 right-10  bg-[#69FF24] text-black px-[75px] py-2 rounded font-bold">
            Order
        </a>
        <div v-if="confirmation" class="text-blue-500 mb-4 text-normal font-bold absolute top-[70px] right-[38px]">{{ confirmation }}</div>
    </div>
</template>