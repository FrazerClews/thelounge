<template>
	<ChannelForm
		v-if="channelData"
		:handle-submit="handleSubmit"
		:defaults="channelData"
		:disabled="disabled"
	/>
</template>

<script>
import socket from "../../js/socket";
import ChannelForm from "../ChannelForm.vue";

export default {
	name: "ChannelEdit",
	components: {
		ChannelForm,
	},
	data() {
		return {
			disabled: false,
			channelData: null,
		};
	},
	watch: {
		"$route.params.id"() {
			this.setChannelData();
		},
	},
	mounted() {
		this.setChannelData();
	},
	methods: {
		setChannelData() {
			socket.emit("channel:get", this.$route.params.id);
			this.channelData = this.$store.getters.findChannel(this.$route.params.id);
		},
		handleSubmit(data) {
			this.disabled = true;
			socket.emit("channel:edit", data);

			// TODO: move channels to vuex and update state when the channel info comes in
			const channel = this.$store.getters.findChannel(data.id);
			channel.name = channel.channels[0].name = data.name;

			this.$root.switchToChannel(channel.channels[0]);
		},
	},
};
</script>
