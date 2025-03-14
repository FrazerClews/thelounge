<template>
	<div id="chat-container" class="window" :data-current-channel="channel.name">
		<div
			id="chat"
			:class="{
				'hide-motd': !$store.state.settings.motd,
				'colored-nicks': $store.state.settings.coloredNicks,
				'time-seconds': $store.state.settings.showSeconds,
				'time-12h': $store.state.settings.use12hClock,
			}"
		>
			<div
				:id="'chan-' + channel.id"
				class="chat-view"
				:data-type="channel.type"
				:aria-label="channel.name"
				role="tabpanel"
			>
				<div class="header">
					<SidebarToggle />
					<span class="title">{{ channel.name }}</span>
					<div v-if="channel.editTopic === true" class="topic-container">
						<input
							ref="topicInput"
							:value="channel.topic"
							class="topic-input"
							placeholder="Set channel topic"
							@keyup.enter="saveTopic"
							@keyup.esc="channel.editTopic = false"
						/>
						<span aria-label="Save topic" class="save-topic" @click="saveTopic">
							<span type="button" aria-label="Save topic"></span>
						</span>
					</div>
					<span v-else :title="channel.topic" class="topic" @dblclick="editTopic"
						><ParsedMessage
							v-if="channel.topic"
							:network="network"
							:text="channel.topic"
					/></span>
					<button
						class="mentions"
						aria-label="Open your mentions"
						@click="openMentions"
					/>
					<button
						class="menu"
						aria-label="Open the context menu"
						@click="openContextMenu"
					/>
					<span
						v-if="channel.type === 'channel'"
						class="rt-tooltip tooltipped tooltipped-w"
						aria-label="Toggle user list"
					>
						<button
							class="rt"
							aria-label="Toggle user list"
							@click="$store.commit('toggleUserlist')"
						/>
					</span>
				</div>
				<div v-if="channel.type === 'special'" class="chat-content">
					<div class="chat">
						<div class="messages">
							<div class="msg">
								<Component
									:is="specialComponent"
									:network="network"
									:channel="channel"
								/>
							</div>
						</div>
					</div>
				</div>
				<div v-else class="chat-content">
					<div
						:class="[
							'scroll-down tooltipped tooltipped-w tooltipped-no-touch',
							{'scroll-down-shown': !channel.scrolledToBottom},
						]"
						aria-label="Jump to recent messages"
						@click="$refs.messageList.jumpToBottom()"
					>
						<div class="scroll-down-arrow" />
					</div>
					<MessageList ref="messageList" :network="network" :channel="channel" />
					<ChatUserList v-if="channel.type === 'channel'" :channel="channel" />
				</div>
			</div>
		</div>
		<div
			v-if="this.$store.state.currentUserVisibleError"
			id="user-visible-error"
			@click="hideUserVisibleError"
		>
			{{ this.$store.state.currentUserVisibleError }}
		</div>
		<ChatInput :network="network" :channel="channel" />
	</div>
</template>

<script>
import socket from "../js/socket";
import eventbus from "../js/eventbus";
import ParsedMessage from "./ParsedMessage.vue";
import MessageList from "./MessageList.vue";
import ChatInput from "./ChatInput.vue";
import ChatUserList from "./ChatUserList.vue";
import SidebarToggle from "./SidebarToggle.vue";
import ListBans from "./Special/ListBans.vue";
import ListInvites from "./Special/ListInvites.vue";
import ListChannels from "./Special/ListChannels.vue";
import ListIgnored from "./Special/ListIgnored.vue";

export default {
	name: "Chat",
	components: {
		ParsedMessage,
		MessageList,
		ChatInput,
		ChatUserList,
		SidebarToggle,
	},
	props: {
		network: Object,
		channel: Object,
	},
	computed: {
		specialComponent() {
			switch (this.channel.special) {
				case "list_bans":
					return ListBans;
				case "list_invites":
					return ListInvites;
				case "list_channels":
					return ListChannels;
				case "list_ignored":
					return ListIgnored;
			}

			return undefined;
		},
	},
	watch: {
		channel() {
			this.channelChanged();
		},
		"channel.editTopic"(newValue) {
			if (newValue) {
				this.$nextTick(() => {
					this.$refs.topicInput.focus();
				});
			}
		},
	},
	mounted() {
		this.channelChanged();

		if (this.channel.editTopic) {
			this.$nextTick(() => {
				this.$refs.topicInput.focus();
			});
		}
	},
	methods: {
		channelChanged() {
			// Triggered when active channel is set or changed
			this.channel.highlight = 0;
			this.channel.unread = 0;

			socket.emit("open", this.channel.id);

			if (this.channel.usersOutdated) {
				this.channel.usersOutdated = false;

				socket.emit("names", {
					target: this.channel.id,
				});
			}
		},
		hideUserVisibleError() {
			this.$store.commit("currentUserVisibleError", null);
		},
		editTopic() {
			if (this.channel.type === "channel") {
				this.channel.editTopic = true;
			}
		},
		saveTopic() {
			this.channel.editTopic = false;
			const newTopic = this.$refs.topicInput.value;

			if (this.channel.topic !== newTopic) {
				const target = this.channel.id;
				const text = `/raw TOPIC ${this.channel.name} :${newTopic}`;
				socket.emit("input", {target, text});
			}
		},
		openContextMenu(event) {
			eventbus.emit("contextmenu:channel", {
				event: event,
				channel: this.channel,
				network: this.network,
			});
		},
		openMentions() {
			eventbus.emit("mentions:toggle", {
				event: event,
			});
		},
	},
};
</script>
