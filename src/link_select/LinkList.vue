<!--

 Copyright (C) 2017  Mephisto


 This program is free software: you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation, either version 3 of the License, or
 (at your option) any later version.

 This program is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 GNU General Public License for more details.

 You should have received a copy of the GNU General Public License
 along with this program.  If not, see <http://www.gnu.org/licenses/>.

-->

<template>
    <div class="table-holder"> <!-- @contextmenu.prevent="$refs.ctxMenu.open">-->
        <el-table
                ref="multipleTable"
                :data=links
                border
                style="width: 100%"
                class="link-table"
                @selection-change="handleSelectionChange"
                @row-click="handleRowClick"
                @row-contextmenu="handleContextMenu">
            <el-table-column
                    type="selection"
                    width="55">
            </el-table-column>
            <el-table-column
                    label="Thumb"
                    class-name="thumbnail-column"
                    width="120">
                <template scope="scope">
                    <!--<img v-if="scope.row.thumb" :src=scope.row.thumb/>-->
                    <el-popover v-if="scope.row.thumb && thumbsEnabled" trigger="hover" placement="right">
                        <img :src=scope.row.thumb />
                        <div slot="reference" class="thumb-wrapper">
                            <img :src=scope.row.thumb />
                        </div>
                    </el-popover>
                    <span class="img-placeholder" v-if="!thumbsEnabled">
                        [hidden]
                    </span>
                </template>
            </el-table-column>
            <el-table-column
                    label="Url"
                    show-overflow-tooltip>
                <template scope="scope">
                    {{ scope.row.url }}
                    <a :href=scope.row.url target="_blank">
                        <i class=" el-icon-document"></i>
                    </a>
                    <i v-if="scope.row.dupe" class="el-icon-warning" title="Link already downloaded or in queue"></i>
                </template>
            </el-table-column>
        </el-table>

        <context-menu id="context-menu" ref="ctxMenu">
            <li class="ctx-item" @click="handleCtxMenuDisable">Disable</li>
            <li class="ctx-item" @click="handleCtxMenuEnable">Enable</li>
            <li class="ctx-item" @click="handleCtxMenuToggle">Toggle</li>
            <li class="ctx-divider"></li>
            <li class="ctx-item" @click="handleCtxMenuToggleThumbs">Toggle thumbnails</li>
        </context-menu>

    </div>
</template>

<script type="text/babel">
    import contextMenu from "vue-context-menu"

    import { SelectionModel } from "../lib/utils";

    export default {
        name: "link-list",
        data() {
            return {
                selectedLinks: [],
                selectionModel: new SelectionModel(() => 1, (groupIdx) => this.links.length),
                thumbsEnabled: true
            };
        },

        mounted() {
            console.debug("[link-select] Mounted", this);

            this.thumbsEnabled = !this.hideThumbs;

            this.$el.querySelector("div.el-table").classList.remove("el-table--enable-row-hover");

            let idx = 0;
            for (let link of this.links) {
                link.index = idx++;
                if (!link.dupe) {
                    this.$refs.multipleTable.toggleRowSelection(link);
                }
            }

            document.addEventListener(
                "keydown",
                (event) => {
                    if (event.key === "a" && event.ctrlKey) {
                        this.clearSelection();
                        this.selectionModel.selectAll(/*includeGroups = */false);
                        this.highlightSelection();

                        event.preventDefault();
                        return true;
                    }
                }
            );
        },

        props: {
            links: {type: Array},
            hideThumbs: {type: Boolean},
            title: {type: String},
        },

        components: { contextMenu },

        methods: {
            handleSelectionChange(val) {
                this.selectedLinks = val;
            },

            getRow(i) {
                let index = parseInt(i) + 1;
                return this.$refs.multipleTable.$el.querySelector(`tbody tr:nth-child(${index})`);
            },

            clearSelection() {
                this.selectionModel.getSelection().forEach((pair) => {
                    this.getRow(pair[1]).style = "";
                });
            },

            highlightSelection() {
                this.selectionModel.getSelection().forEach((pair) => {
                    this.getRow(pair[1]).style = "background-color: lightsteelblue";
                });
            },

            handleRowClick(row, event, column) {
                this.clearSelection();

                let index = row.index;

                if (!event.shiftKey && !event.ctrlKey) {
                    this.selectionModel.selectSingle(0, index);
                }

                if (event.ctrlKey) {
                    this.selectionModel.selectAppend(0, index);
                }

                if (event.shiftKey) {
                    this.selectionModel.selectRange(0, index);
                }

                this.highlightSelection();
            },

            handleContextMenu(row, event) {
                if (!this.selectionModel.isSelected(0, row.index)) {
                    this.clearSelection();
                    this.selectionModel.selectSingle(0, row.index);
                    this.highlightSelection();
                }
                this.$refs.ctxMenu.open(event);
                event.preventDefault();
            },

            toggleSelection(toggle) {
                for (let selection in this.selectionModel.selection) {
                    let rowIdx = selection.split(":")[1];
                    this.$refs.multipleTable.toggleRowSelection(this.links[rowIdx], toggle);
                }
            },

            handleCtxMenuDisable() {
                this.toggleSelection(false);
            },

            handleCtxMenuEnable() {
                this.toggleSelection(true);
            },

            handleCtxMenuToggle() {
                this.toggleSelection();
            },

            handleCtxMenuToggleThumbs() {
                this.thumbsEnabled = !this.thumbsEnabled;
                /*this.$refs.multipleTable.$el.querySelectorAll(".thumbnail-column img").forEach((e) => {
                    e.style.display = e.style.display ? "" : "none";
                });*/
            }
        }
    }
</script>

<style>
    .cell img {
        max-width: 100%;
        padding: 5px;
        vertical-align: center;
    }

    .thumbnail-column {
        text-align: center;
    }
    
    .link-table {
        -moz-user-select: none;
    }
</style>
