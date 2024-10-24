module EducationBonds::BondModule {

    use aptos_framework::signer;
    use aptos_framework::coin;
    use aptos_framework::aptos_coin::AptosCoin;

    /// Struct representing an education bond.
    struct Bond has store, key {
        institution: address,    // Address of the educational institution
        total_issued: u64,       // Total bonds issued
        repaid: u64,             // Total amount repaid by students
    }

    /// Function to issue education bonds for an institution.
    public fun issue_bonds(institution: &signer, amount: u64) {
        let bond = Bond {
            institution: signer::address_of(institution),
            total_issued: amount,
            repaid: 0,
        };
        move_to(institution, bond);
    }

    /// Function for students to repay the bond via student fees.
    public fun repay_bond(student: &signer, institution: address, amount: u64) acquires Bond {
        let bond = borrow_global_mut<Bond>(institution);

        // Transfer repayment from the student to the institution
        let payment = coin::withdraw<AptosCoin>(student, amount);
        coin::deposit<AptosCoin>(institution, payment);

        // Update the repaid amount
        bond.repaid = bond.repaid + amount;
    }
}
