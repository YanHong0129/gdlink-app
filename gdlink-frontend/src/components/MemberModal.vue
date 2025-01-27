<template>
    <div>
      <div 
        class="modal fade" 
        id="memberListModal" 
        data-bs-backdrop="static" 
        data-bs-keyboard="false" 
        tabindex="-1" 
        aria-labelledby="memberListLabel" 
        aria-hidden="true"
      >
        <div class="modal-dialog modal-dialog-centered modal-lg">
          <div class="modal-content">
            <div class="modal-header">
              <h2 class="modal-title w-100 text-center" id="memberListLabel">
                {{ group.groupName }}
              </h2>
              <button 
                type="button" 
                class="btn-close position-absolute" 
                style="right: 20px;" 
                data-bs-dismiss="modal" 
                aria-label="Close"
              ></button>
            </div>
            <div class="modal-body p-4">
                <table class="table table-bordered">
                    <thead>
                        <tr>
                            <th scope="col">No.</th>
                            <th scope="col">Member Name</th>
                            <th scope="col">Member Email</th>
                            <th scope="col">Action</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(member, index) in groupMembers" :key="member.groupMemberId">
                            <td style="width:5%" scope="row">{{index+1}}</td>
                            <td :style="{ width: '50%', color: member.name ? 'inherit' : 'red' }">
                                {{ member.name || 'Unknown' }}
                            </td>                           
                            <td style="width:35%">{{ member.memberEmail }}</td>
                            <td style="width:10%">
                                <div class="d-flex flex-row gap-3">
                                    <span @click="removeMember(member.groupMemberId)">
                                        <i class="bi bi-trash3"></i>
                                    </span>
                                </div>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <div class="modal-footer">
                <small v-if="errors.memberEmail" class="text-danger">{{ errors.memberEmail }}</small>
                <input 
                    type="text" 
                    class="form-control"
                    placeholder="Enter User Email"
                    id="memberEmail" 
                    v-model="memberEmail" 
                />
                <button @click="submit()" class="btn btn-outline-primary">Add Member</button>
            </div>
          </div>
        </div>
      </div>
    </div>
</template>

<script>
import GroupMemberService from '../service/GroupMemberService';
import Swal from 'sweetalert2';
import SweetAlert from '@/Utils/SweetAlertUtils';

export default {
    data() {
        return {
            group: [],
            groupMembers: [],
            memberEmail: null,
            userId: null,
            errors: {
                memberEmail: ''
            }
        };
    },
    methods:{
        validateEmail() {
            this.errors = {
                memberEmail: ''
            };

            let isValid = true;

            const emailPattern = /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
            if (!this.memberEmail) {
                this.errors.memberEmail = "Email is required.";
                isValid = false;
                } else if (!emailPattern.test(this.memberEmail)) {
                this.errors.memberEmail = "Please enter a valid email address.";
                isValid = false;
            }

            return isValid;
        },
        openModalForMembers(group) {
            this.group = group;
            this.displayMemberList();
            this.resetForm();
        },
        async displayMemberList(){
            this.groupMembers = await GroupMemberService.getMemberList(this.group.groupId);
        },
        async submit(){
            if(this.validateEmail()){
                await this.addMember();
                this.displayMemberList();
                this.resetForm();
            }
        },
        resetForm() {
            this.memberEmail = null;
        },
        async addMember(){
            try {
                const data = await GroupMemberService.addMember(this.group.groupId,this.memberEmail);
                if (data.success) {
                    SweetAlert.showSwal('Added!', data.message, 'success');
                } else {
                    SweetAlert.showSwal('Error!', data.message, 'error');
                }
            } catch (error) {
                console.error('Error adding member:', error);
                SweetAlert.showSwal('Error!', 'An unexpected error occurred. Please try again later.', 'error');
            }
            this.$emit('refresh');
        },
        async updateMemberRole(member){
            try {
                const data = await GroupMemberService.updateMemberRole(member);
                if (data.success) {
                Swal.fire({
                    title: 'Updated!',
                    text: data.message,
                    icon: 'success',
                    timer: 2000,
                    showConfirmButton: false,
                });
                } else {
                    Swal.fire({
                        title: 'Error!',
                        text: data.message,
                        icon: 'error',
                        timer: 2000,
                        showConfirmButton: false,
                    });
                }
                this.displayMemberList();
            } catch (error) {
                    console.error('Error updating role:', error);
                    Swal.fire({
                        title: 'Error!',
                        text: 'An unexpected error occurred. Please try again later.',
                        icon: 'error',
                        timer: 2000,
                        showConfirmButton: false,
                    });
            }
        },
        async removeMember(groupMemberId){
            await SweetAlert.deleteSwal({
                confirmTitle: 'Are you sure?',
                confirmText: 'This action will permanently remove the member.',
                confirmButtonText: 'Yes, remove!',
                cancelButtonText: 'Cancel',
                successTitle: 'Removed!',
                successText: 'The member has been removed.',
                errorTitle: 'Error!',
                errorText: 'Failed to remove the member. Please try again.',
                unexpectedErrorText: 'An unexpected error occurred. Please try again later.',
                deleteAction: () => GroupMemberService.removeMember(groupMemberId),
                refreshData: () => this.displayMemberList(),
            });
            this.$emit('refresh');
        }
        
    }
};
</script>

<style scoped>
.modal-footer{
    flex-direction: row;
    flex-wrap: nowrap;
}

input{
    width:30%;
}

.btn{
    width: 120px;
}

td{
    vertical-align: middle;
}

td div span{
    cursor: pointer;
    font-size: 20px;
    padding: 5px;
    height: 30px;
    width: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 25px;
}

td div span i{
    transform: translateY(-0.5px);
}

td div span:hover{
    background-color: rgb(160, 206, 247);
}

td div span:active{
    background-color: rgb(160, 206, 247);
    transform:scale(0.9);
}

.bi-trash3{
    color: rgb(194, 0, 0);
}



</style>