<template>
	<div id="channel" class="window" role="tabpanel" aria-label="Channel">
		<div class="header">
			<SidebarToggle />
		</div>
		<form class="container" method="post" action="" @submit.prevent="onSubmit">
			<h1 class="title">
				<template>
					<input v-model="defaults.id" type="hidden" name="id" />
					Edit {{ defaults.name }} Channel
				</template>
			</h1>
			<template>
				<h2>Channel settings</h2>
				<div class="channel-row">
					<label for="channel:name">Name</label>
					<input
						id="channel:name"
						v-model="defaults.name"
						class="input"
						name="name"
						maxlength="100"
					/>
				</div>
				<div class="channel-row">
					<label for="channel:password">Password</label>
					<RevealPassword
						v-slot:default="slotProps"
						class="input-wrap password-container"
					>
						<input
							id="channel:password"
							v-model="defaults.password"
							class="input"
							:type="slotProps.isVisible ? 'text' : 'password'"
							placeholder="Server password (optional)"
							name="password"
							maxlength="300"
						/>
					</RevealPassword>
				</div>
				<div class="channel-row">
					<label for="channel:topic">Topic</label>
					<input
						id="channel:topic"
						v-model="defaults.topic"
						class="input"
						name="name"
						maxlength="512"
					/>
				</div>
				<div class="channel-row">
					<label>Mute Channel</label>
					<div class="input-wrap">
						<label class="mute">
							<input v-model="defaults.mute" type="checkbox" name="mute" />
						</label>
					</div>
				</div>
			</template>

			<!--
			<h2>User preferences</h2>

			<template>
				<div class="channel-row">
					<button type="submit" class="btn" :disabled="disabled ? true : false">
						<template v-if="defaults.id">Save channel</template>
						<template v-else>Channel</template>
					</button>
				</div>
			</template>
			-->
		</form>
	</div>
</template>

<style>
#channel .channel-auth {
	display: block;
	margin-bottom: 10px;
}

#channel .channel-auth .opt {
	display: block;
	width: 100%;
}

#channel .channel-auth input {
	margin: 3px 10px 0 0;
}

#channel .channel-sasl-external {
	padding: 10px;
	border-radius: 2px;
	background-color: #d9edf7;
	color: #31708f;
}

#channel .channel-sasl-external pre {
	margin: 0;
	user-select: text;
}
</style>

<script>
import socket from "../../js/socket";
import SidebarToggle from "../SidebarToggle.vue";

export default {
	name: "ChannelEdit",
	components: {
		SidebarToggle,
	},
	props: {
		defaults: Object,
		disabled: Boolean,
	},
	data() {
		return {
			channel: this.channel,
		};
	},
	watch: {
		"$route.params.id"() {
			this.setChannelData();
		},

		displayPasswordField(value) {
			if (value) {
				this.$nextTick(() => this.$refs.publicPassword.focus());
			}
		},
		"defaults.commands"() {
			this.$nextTick(this.resizeCommandsInput);
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
		onSubmit(event) {
			const formData = new FormData(event.target);
			const data = {};

			for (const item of formData.entries()) {
				data[item[0]] = item[1];
			}

			this.disabled = true;
			socket.emit("channel:edit", data);

			// TODO: move channels to vuex and update state when the channel info comes in
			const channel = this.$store.getters.findChannel(data.id);
			channel.name = channel.channels[0].name = data.name;

			this.$root.switchToChannel(channel.channels[0]);
		},
		resizeCommandsInput() {
			if (!this.$refs.commandsInput) {
				return;
			}

			// Reset height first so it can down size
			this.$refs.commandsInput.style.height = "";

			// 2 pixels to account for the border
			this.$refs.commandsInput.style.height =
				Math.ceil(this.$refs.commandsInput.scrollHeight + 2) + "px";
		},
	},
};
</script>
