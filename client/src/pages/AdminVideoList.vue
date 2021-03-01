<template>
  <q-page class="row items-start q-pa-lg">
    <q-table 
      class="full-width"
      :columns="columns"
      :rows="rows"
      :rows-per-page-options="[10, 20, 0]"
    >
      <template #top-right>
        <q-btn @click="dialog = true">
          {{ $t('addNew') }}
        </q-btn>
      </template>
      
      <template #body-cell-actions="props">
        <q-td :props="props">
          <q-icon
            class="q-mr-sm"
            name="visibility"
            @click="viewVideo(props.row)"
          />
          <q-icon
            name="delete"
            @click="deleteVideo(props.row)"
          />
        </q-td>
      </template>
    </q-table>

    <q-dialog v-model="dialog">
      <q-card
        class="full-width"
        style="max-width:500px"
      >
        <q-card-section class="q-gutter-md">
          <q-input
            v-model="name"
            :label="$t('name')"
          />

          <q-file 
            v-model="file"
            accept=".mp4"
            filled
            bottom-slots
            :label="$t('file')"
            counter
          >
            <template #prepend>
              <q-icon
                name="cloud_upload"
                @click.stop
              />
            </template>
            <template #append>
              <q-icon
                name="close"
                class="cursor-pointer"
                @click.stop="file = null"
              />
            </template>
          </q-file>
        </q-card-section>

        <q-card-actions class="justify-end">
          <q-btn
            :disabled="!name || !file"
            color="primary" 
            @click="addVideo"
          >
            {{ $t("upload") }}
          </q-btn>
          <q-btn @click="dialog = false">
            {{ $t("cancel") }}
          </q-btn>
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue';

import { useI18n } from 'vue-i18n';
import { useRouter } from 'vue-router';
import { api  } from 'boot/axios';
import { Video } from 'src/types/video';

export default defineComponent({
    name: 'PageIndex',
    components: {
 
    },
    setup() {
        const tm = useI18n();
        const router = useRouter();

        const rows = ref<Video[]>([]);
        const dialog = ref(false);
        const file = ref<File>();
        const name = ref('');

        const columns = [
            {
                label: tm.t('name'),
                name: 'name',
                field: 'name',
                align: 'left'
            },
            {
                label: tm.t('directory'),
                name: 'directory',
                field: 'path',
                align: 'left'
            },
            {
                name: 'actions', 
            },
        ];

        const setVideos = async () => {
            const { data } = await api.get<Video[]>('admin/videos');
            rows.value = data;
        };

        onMounted(setVideos);

        const addVideo = async () => {

            if (!file.value) {
                return;
            }

            const formData = new FormData();

            formData.append('name', name.value);
            formData.append('file', file.value);

            await api.post('admin/videos', formData);

            await setVideos();

            dialog.value = false;
        };
    
        const viewVideo = async (item: Video) => {
            await router.push({
                name: 'admin-video',
                params: {
                    videoId: item.id 
                }
            });
        };
    
        const deleteVideo = async (item: Video) => {
            await api.delete(`videos/${item.id}`);
            await setVideos();
        };
    
        return {
            name,
            file,
            rows,
            columns,
            dialog,
            addVideo,
            viewVideo,
            deleteVideo
        };
    }
});
</script>