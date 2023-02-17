import { LightningElement, api } from 'lwc';
import generatePDF from '@salesforce/apex/QRA_WaiverController.generatePDF';
import getUserDetails from '@salesforce/apex/QRA_WaiverController.getUserDetails';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import { CloseActionScreenEvent } from 'lightning/actions';

export default class qraWaiverCmp extends LightningElement {
    attachment; //this will hold attachment reference

    @api recordId;
    @api objectApiName;
    boolShowSpinner = false;
    selectedWaivers;
    textBox1;
    textBox2;
    complianceUserName;
    todayDate;
    
    connectedCallback(){
        console.log('entered');
        getUserDetails()
        .then((result)=> {
            this.complianceUserName = result.complianceUserName;
            this.todayDate = result.todayDate;
            console.log('ee',this.complianceUserName, this.todayDate);
            })
        .catch(error=>{
                console.log('error',error);
                //show error message
                this.dispatchEvent(
                    new ShowToastEvent({
                        title: 'Error creating Attachment record',
                        message: error.body.message,
                        variant: 'error',
                    }),
                );
            })
    }

    saveAsPdf(){
        this.boolShowSpinner = true;
        this.selectedWaivers = [...this.template.querySelectorAll('lightning-input')].filter(element => element.checked).map(element => element.dataset.id);
        if(this.textBox1 == undefined){this.textBox1 = '';}
        if(this.textBox2 == undefined){this.textBox2 = '';}
        if(this.selectedWaivers.length >0){
            generatePDF({selectedWaivers:this.selectedWaivers, departmentId:this.recordId, objectApiName: this.objectApiName, textBox1: this.textBox1, textBox2: this.textBox2})
            .then((result)=>{
                    //show success message
                    this.dispatchEvent(
                        new ShowToastEvent({
                            title: 'Success',
                            message: 'Waiver successfully generated',
                            variant: 'success',
                        }),
                    );
                    //refresh Files related list to reflect the newly created file on record
                    eval("$A.get('e.force:refreshView').fire();");
                    //close the quick action modal
                    this.dispatchEvent(new CloseActionScreenEvent());
            })
            .catch(error=>{
                console.log('error',error);
                //show error message
                this.dispatchEvent(
                    new ShowToastEvent({
                        title: 'Error creating Attachment record',
                        message: error.body.message,
                        variant: 'error',
                    }),
                );
            })
        }else{
            this.boolShowSpinner = false;
            this.dispatchEvent(
                new ShowToastEvent({
                    title: 'Error',
                    message: 'Please select atleast one option',
                    variant: 'error',
                }),
            );
        }
        
    }

    //Select parent checkboxes when child is selected
    autoSelectParent(event){
       var selectedCheckBox = event.target.dataset.id;
       //check if checkbox is valid child
       if(selectedCheckBox.startsWith('PN_')){
          if(event.target.checked){
            this.template.querySelector('[data-id="PN"]').checked = true;
          }
       }
       if(selectedCheckBox.startsWith("PE_")){
        if(event.target.checked){
            this.template.querySelector('[data-id="PE"]').checked = true;
        }
       }
       if(selectedCheckBox.startsWith("RE_")){
        if(event.target.checked){
            this.template.querySelector('[data-id="RE"]').checked = true;
        }
       }
       if(selectedCheckBox.startsWith("other_")){
        if(event.target.checked){
            this.template.querySelector('[data-id="other"]').checked = true;
        }
       }
    }

    captureTextFields(event){
        if(event.target.name == 'textBox1'){
            this.textBox1 = event.target.value;
        }else{
            this.textBox2 = event.target.value;
        }
    }
 
    renderedCallback() {
		const style = document.createElement("style");
		style.innerText ='.custom-waiver-spinner .slds-spinner{position:fixed ;} .font-size-16 .slds-checkbox .slds-checkbox__label .slds-form-element__label{font-size:1rem}.font-size-16 .slds-min-width .slds-checkbox .slds-checkbox__label{display:flex;align-items:center} .font-size-16 .slds-min-width .slds-checkbox .slds-checkbox__label .slds-form-element__label{min-width: 376px;}';

		if (this.template.querySelector(".qra-waiver-modal"))
		  this.template.querySelector(".qra-waiver-modal").appendChild(style);
	}   

}
