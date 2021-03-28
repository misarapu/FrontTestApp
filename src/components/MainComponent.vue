<template>
	<v-container>
		<v-row>
			<v-col md="6">
				<v-card class="pa-10" outlined>
					<h3>
						Please enter your name and pick the Sectors you are currently
						involved in.
					</h3>
					<v-form
						ref="form"
						action="/"
						v-model="valid"
						@submit="loginSector"
						method="post"
						lazy-validation
					>
						<v-text-field
							v-model="name"
							:counter="20"
							:rules="nameRules"
							label="Name"
							required
						></v-text-field>

						<v-select
							v-model="selectedSector"
							:rules="[(v) => !!v || 'Item is required']"
							:items="sectors"
              item-value="id"
							label="Sector"
							required
						>
							<template v-slot:item="{ item }">
								<v-list-item-content
									:class="{
										'pl-2': item.rank === 1,
										'pl-6': item.rank === 2,
										'pl-10': item.rank === 3,
										'pl-14': item.rank === 4,
									}"
								>
									{{ item.name }}
								</v-list-item-content>
							</template>
							<template v-slot:selection="{ item }">
								{{ item.name }}
							</template>
						</v-select>

						<v-checkbox
							v-model="checkbox"
							:rules="[(v) => !!v || 'You must agree to continue!']"
							label="Agree to terms"
							required
						></v-checkbox>

						<v-btn
							:disabled="!valid"
							color="success"
							class="mr-4"
							type="submit"
							@click="validate"
						>
							Save
						</v-btn>
					</v-form>
				</v-card>
			</v-col>
		</v-row>
	</v-container>
</template>

<script>
export default {
	data: () => ({
    root: "http://localhost:3000/api/",
		valid: true,
		name: "",
		nameRules: [
			(v) => !!v || "Name is required",
			(v) => (v && v.length <= 20) || "Name must be less than 20 characters",
		],

		checkbox: false,
		selectedSector: null,
		sectors: [],
	}),

	methods: {
		validate() {
			this.$refs.form.validate();
		},
		sectorRank(sector) {
			if (sector.group_2 === 0 && sector.group_3 === 0 && sector.group_4 === 0)
				return 1;
			if (sector.group_3 === 0 && sector.group_4 === 0) return 2;
			if (sector.group_4) return 3;
			return 4;
		},
		loginSector() {
			this.axios
				.post(this.root + "logins", {
					user_name: this.name,
					sector_id: this.selectedSector
				})
				.catch((err) => console.log("Posting error: " + err.message));
		},
	},

	mounted() {
		this.axios
			.get(this.root + "sectors")
			.then((res) => {
				this.sectors = res.data.map((obj) => ({
					...obj,
					rank: this.sectorRank(obj),
				}));
			})
			.catch((err) => console.log("Sectors fetching error: " + err.message));
	},
};
</script>
<style scoped>
div.sectors-container {
	height: 300px;
	overflow-y: scroll;
	cursor: pointer;
}

div.sectors-container p {
	margin-bottom: 0;
}
div.sectors-container p:hover {
	background-color: #90a4ae;
}

p.selected {
	background-color: #455a64 !important;
	color: white;
	font-weight: bold;
}
</style>