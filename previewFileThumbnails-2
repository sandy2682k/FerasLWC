************ FileController.cls **************
public with sharing class FileController {
    @AuraEnabled(cacheable=true)
    public static List<ContentVersion> getVersionFiles(String recordId) {
        try {
            System.debug('getVersionFiles: recordId = ' + recordId);
            List<ContentVersion> versions = [
                SELECT
                    Id,
                    Title,
                    ContentDocumentId,
                    FileType, 
                    ContentSize,
                    FileExtension,
                    VersionNumber,
                    CreatedDate,
                    VersionData,
                    FirstPublishLocationId
                FROM ContentVersion
                WHERE FirstPublishLocationId = :recordId
                ORDER BY CreatedDate DESC
            ];
            System.debug('getVersionFiles: Found ' + versions + ' Content Versions');
            return versions;
        } catch (Exception e) {
            System.debug('getVersionFiles: Exception - ' + e.getMessage());
            throw new AuraHandledException(e.getMessage());
        }
    }
    
    @AuraEnabled
    public static void createNoteRecord(String recordId, String note) {
        try {
            System.debug('createNoteRecord: recordId = ' + recordId);
            ContentVersion contentVersionRecord = [SELECT Id, Description FROM ContentVersion WHERE Id = :recordId LIMIT 1];
            if (contentVersionRecord != null) {
                System.debug('createNoteRecord: Updating Content Version Description');
                contentVersionRecord.Description = note;
                update contentVersionRecord;
                System.debug('createNoteRecord: Content Version Updated');
            } else {
                System.debug('createNoteRecord: ContentVersion record not found.');
                throw new AuraHandledException('ContentVersion record not found.');
            }
        } catch (Exception e) {
            System.debug('createNoteRecord: Exception - ' + e.getMessage());
            throw new AuraHandledException(e.getMessage());
        }
    }

    @AuraEnabled(cacheable= true)
    public static String getNote(String recordId ) {
        String note = '';
        try {
            System.debug('getNote: recordId = ' + recordId);
            ContentVersion contentVersionRecord = [SELECT Id, Description FROM ContentVersion WHERE Id = :recordId ];
            note = contentVersionRecord.Description;
            System.debug('getNote: Found note - ' + note);
        } catch (Exception e) {
            System.debug('getNote: Exception - ' + e.getMessage());
            throw new AuraHandledException(e.getMessage());
        }
        return note;
    }
}

******************** previewFileThumbnails.html *****************

<template>
  <lightning-card title="Uploaded Files">
    <lightning-file-upload
      name="fileUploader"
      accept={acceptedFormats}
      record-id={recordId}
      onuploadfinished={handleUploadFinished}
      multiple
    ></lightning-file-upload>

    <div class="slds-grid slds-wrap slds-gutters">
      <template if:true={loaded}>
        <template for:each={files} for:item="file">
          <div key={file.Id} class="slds-size_1-of-2 slds-p-around_medium">
            <c-preview-file-thumbnail-card
              key={file.Id}
              file={file}
              record-id={recordId}
              thumbnail={file.thumbnailFileCard}
            ></c-preview-file-thumbnail-card>

            <c-file-details
              key={file.Id}
              file={file}

            ></c-file-details>
          </div>
        </template>
      </template>
    </div>
  </lightning-card>
</template>

******************** previewFileThumbnails.js *****************
import { LightningElement, wire, api, track } from "lwc";
import { refreshApex } from "@salesforce/apex";
import { ShowToastEvent } from "lightning/platformShowToastEvent";
import getFileVersions from "@salesforce/apex/FileController.getVersionFiles";

export default class PreviewFileThumbnails extends LightningElement {
  loaded = false;
  @track fileList;
  @api recordId;
  @track files = [];
  note = ""; // Added text area value
  fetchedNote = ""; //
  get acceptedFormats() {
    return [".pdf", ".png", ".jpg", ".jpeg"];
  }

  @wire(getFileVersions, { recordId: "$recordId" })
  fileResponse(value) {
    this.wiredActivities = value;
    const { data, error } = value;
    this.fileList = "";
    this.files = [];
    if (data) {
      this.fileList = data;
      for (let i = 0; i < this.fileList.length; i++) {
        let file = {
          Id: this.fileList[i].Id,
          Title: this.fileList[i].Title,
          Extension: this.fileList[i].FileExtension,
          ContentDocumentId: this.fileList[i].ContentDocumentId,
          ContentDocument: this.fileList[i].ContentDocument,
          CreatedDate: this.fileList[i].CreatedDate,
          thumbnailFileCard:
            "/sfc/servlet.shepherd/version/renditionDownload?rendition=THUMB720BY480&versionId=" +
            this.fileList[i].Id +
            "&operationContext=CHATTER&contentId=" +
            this.fileList[i].ContentDocumentId,
          downloadUrl:
            "/sfc/servlet.shepherd/document/download/" +
            this.fileList[i].ContentDocumentId
        };
        this.files.push(file);
      }
      this.loaded = true;
    } else if (error) {
      this.dispatchEvent(
        new ShowToastEvent({
          title: "Error loading Files",
          message: error.body.message,
          variant: "error"
        })
      );
    }
  }
  handleUploadFinished(event) {
    const uploadedFiles = event.detail.files;
    refreshApex(this.wiredActivities);
    this.dispatchEvent(
      new ShowToastEvent({
        title: "Success!",
        message: uploadedFiles.length + " Files Uploaded Successfully.",
        variant: "success"
      })
    );
  }
}

*********** fileDetails.html ***********

<template>
<div class="slds-m-around_medium container">
    <lightning-card title={file.Title} icon-name="standard:file">
        <div class="slds-grid slds-gutters">
            <div class="slds-col slds-size_3-of-4 textarea-container">
                <lightning-textarea
                label="Note And description"
                value={fetchedNote}
                onchange={handleNoteChange}
                data-id={file.Id}
            ></lightning-textarea>
            </div>
            <div style="margin-top: 30px;">
                <lightning-button
                    label="Add Note"
                    onclick={handleSaveNote}
                    data-id={file.Id}
                    variant="brand"
                ></lightning-button>
            </div>
        </div>
    </lightning-card>
</div>
</template>

*********** fileDetails.js **************

import { LightningElement, api  , track ,wire} from "lwc";
import { ShowToastEvent } from "lightning/platformShowToastEvent";
import createNoteRecord from "@salesforce/apex/FileController.createNoteRecord";
import getNote from "@salesforce/apex/FileController.getNote";

export default class FileDetails extends LightningElement {
  @api file;
  @api key;
  @track noteValue;
  @track fetchedNote;

  handleNoteChange(event) {
    console.log('Event' , event);
    console.log('Handle text box change');
    this.noteValue = event.target.value;
    console.log('Note Value' , this.noteValue );
    const fileId = event.target.dataset.id;
    console.log('field Id' , fileId);
  }

  connectedCallback(){
    console.log('Files from Parent' , JSON.stringify(this.file));
    console.log('key' , this.key);
  }

  handleSaveNote(event) {
    console.log('Save Note');
    const fileId = event.target.dataset.id;
    console.log('Id on save ' , fileId);

    createNoteRecord({ recordId: fileId, note: this.noteValue  })
      .then(() => {
        this.dispatchEvent(
          new ShowToastEvent({
            title: "Success!",
            message: "Note saved successfully.",
            variant: "success",
          })
        );
      })
      .catch((error) => {
        this.dispatchEvent(
          new ShowToastEvent({
            title: "Error",
            message: error.body.message,
            variant: "error",
          })
        );
      });
  }

  // Call the getNote method using @wire
  @wire(getNote , {recordId: '$file.Id'})
  wiredNote({ error, data }) {
    if (data) {
      this.fetchedNote = data;
    } else if (error) {
      console.error("Error fetching note", error);
    }
  }
}

*********** previewFileModal.html *************

<template>
    <template if:true={showModal}>
      <section
        role="dialog"
        tabindex="-1"
        aria-labelledby="modal-heading-01"
        aria-modal="true"
        aria-describedby="modal-content-id-1"
        class="slds-modal slds-fade-in-open"
      >
        <div class="slds-modal__container">
          <!-- Header Start -->
          <header class="slds-modal__header">
            <lightning-button-icon
              class="slds-modal__close"
              title="Close"
              icon-name="utility:close"
              icon-class="slds-button_icon-inverse"
              onclick={closeModal}
            ></lightning-button-icon>
  
            <h2 id="id-of-modalheader-h2" class="slds-text-heading_large">
              File Preview
            </h2>
          </header>
          <!-- Header End -->
          <div class="slds-modal__content" id="modal-content-id-1">
            <lightning-layout>
              <lightning-layout-item flexibility="auto">
                <article class="slds-card">
                  <div
                    class="slds-card__body slds-card__body_inner"
                    style="margin: 0"
                  >
                    <template if:false={showFrame}>
                      <img src={url} />
                    </template>
                    <template if:true={showFrame}>
                      <iframe
                        src={url}
                        style="width: 100%; height: 800px"
                      ></iframe>
                    </template>
                  </div>
                </article>
              </lightning-layout-item>
            </lightning-layout>
          </div>
          <footer class="slds-modal__footer slds-grid slds-grid_align-spread">
            <lightning-button
              variant="brand-outline"
              label="Cancel"
              onclick={closeModal}
              title="Cancel"
              class="slds-var-m-left_x-small"
            ></lightning-button>
          </footer>
        </div>
      </section>
      <div class="slds-backdrop slds-backdrop_open"></div>
    </template>
  </template>

*********** PreviewFileModal.css **********

.slds-modal__container {
    width: 90% !important;
    max-width: 90% !important;
  }

********** previewFileModal.js ************
import { LightningElement, api } from "lwc";

export default class PreviewFileModal extends LightningElement {
  @api url;
  @api fileExtension;
  showFrame = false;
  showModal = false;
  @api show() {
    console.log("###showFrame : " + this.fileExtension);
    if (this.fileExtension === "pdf") this.showFrame = true;
    else this.showFrame = false;
    this.showModal = true;
  }
  closeModal() {
    this.showModal = false;
  }
}

********** previewFileThumbnailCard.html **********

<template>
  <template if:true={file}>
    <c-preview-file-modal
      url={file.downloadUrl}
      file-extension={file.Extension}
    ></c-preview-file-modal>
    <li
      class="
        slds-col
        slds-var-p-vertical_x-small
        slds-small-size_1-of-2
        slds-medium-size_1-of-4
        slds-large-size_1-of-6
      "
    >
      <div
        class="slds-file slds-file_card slds-has-title"
        style="width: 14rem"
        onclick={filePreview}
      >
        <figure>
          <a class="slds-file__crop slds-file__crop_4-by-3 slds-m-top_x-small">
            <img src={thumbnail} alt={file.title} onclick={filePreview} />
          </a>
          <figcaption class="slds-file__title slds-file__title_card">
            <div class="slds-media slds-media_small slds-media_center">
              <lightning-icon
                icon-name={iconName}
                alternative-text={file.Title}
                title={file.Title}
                size="xx-small"
              >
                <span class="slds-assistive-text"
                  >{file.Title}.{file.Extension}</span
                >
              </lightning-icon>
              <div class="slds-media__body slds-var-p-left_xx-small">
                <span class="slds-file__text slds-truncate" title={file.Title}
                  >{file.Title}.{file.Extension}</span
                >
              </div>
            </div>
          </figcaption>
        </figure>

        <div class="slds-is-absolute" style="top: 3px; right: 5px; z-index: 10">
          <!-- <lightning-button-menu
            alternative-text="Show File Menu"
            variant="border-filled"
            icon-size="xx-small"
          >
            <lightning-menu-item
              value="preview"
              label="Preview"
            ></lightning-menu-item>
            <lightning-menu-item
              value="delete"
              label="Delete"
            ></lightning-menu-item>
          </lightning-button-menu> -->
        </div>
      </div>
    </li>
  </template>
</template>

************** previewFileThumbnailCard.js *********

import { LightningElement, api } from "lwc";

export default class PreviewFileThumbnailCard extends LightningElement {
  @api file;
  @api recordId;
  @api thumbnail;

  get iconName() {
    if (this.file.Extension) {
      if (this.file.Extension === "pdf") {
        return "doctype:pdf";
      }
      if (this.file.Extension === "ppt") {
        return "doctype:ppt";
      }
      if (this.file.Extension === "xls") {
        return "doctype:excel";
      }
      if (this.file.Extension === "csv") {
        return "doctype:csv";
      }
      if (this.file.Extension === "txt") {
        return "doctype:txt";
      }
      if (this.file.Extension === "xml") {
        return "doctype:xml";
      }
      if (this.file.Extension === "doc") {
        return "doctype:word";
      }
      if (this.file.Extension === "zip") {
        return "doctype:zip";
      }
      if (this.file.Extension === "rtf") {
        return "doctype:rtf";
      }
      if (this.file.Extension === "psd") {
        return "doctype:psd";
      }
      if (this.file.Extension === "html") {
        return "doctype:html";
      }
      if (this.file.Extension === "gdoc") {
        return "doctype:gdoc";
      }
    }
    return "doctype:image";
  }

  filePreview() {
    console.log("###Click");
    const showPreview = this.template.querySelector("c-preview-file-modal");
    showPreview.show();
  }
}

