class TDSCalculator:

    def __init__(self):
        self.tds_master = {
            "professional_fees": {"rate": 0.10, "threshold": 30000},
            "contract_individual": {"rate": 0.01, "threshold": 30000, "annual_limit": 100000},
            "contract_others": {"rate": 0.02, "threshold": 30000, "annual_limit": 100000},
            "commission": {"rate": 0.05, "threshold": 15000},
            "interest": {"rate": 0.10, "threshold": 40000}
        }

    def get_applicable_rate(self, base_rate, pan_available, specified_person):
        rate = base_rate

        # 206AA
        if not pan_available:
            rate = max(rate, 0.20)

        # 206AB
        if specified_person:
            rate = max(rate * 2, 0.05)

        return rate

    def calculate_tds(self,
                      nature,
                      amount,
                      aggregate_amount=0,
                      monthly_rent=None,
                      pan_available=True,
                      specified_person=False,
                      surcharge_rate=0,
                      cess_rate=0.04):

        # ================= RENT LOGIC =================
        if nature == "rent":
            if monthly_rent is None:
                raise Exception("Monthly rent required for rent calculation")

            if monthly_rent <= 50000:
                return {
                    "tds": 0,
                    "status": "No deduction - rent ≤ 50,000 per month"
                }

            rate = self.get_applicable_rate(0.10, pan_available, specified_person)

            base_tds = amount * rate
            surcharge = base_tds * surcharge_rate
            cess = (base_tds + surcharge) * cess_rate
            total_tds = base_tds + surcharge + cess

            return {
                "nature": "rent",
                "monthly_rent": monthly_rent,
                "amount": amount,
                "rate": rate,
                "base_tds": round(base_tds, 2),
                "surcharge": round(surcharge, 2),
                "cess": round(cess, 2),
                "total_tds": round(total_tds, 2)
            }

        # ================= OTHER PAYMENTS =================
        if nature not in self.tds_master:
            raise Exception("Invalid nature of payment")

        config = self.tds_master[nature]

        if amount < config["threshold"]:
            return {
                "tds": 0,
                "status": "No deduction - below threshold"
            }

        if "annual_limit" in config:
            if aggregate_amount < config["annual_limit"]:
                return {
                    "tds": 0,
                    "status": "No deduction - below annual limit"
                }

        rate = self.get_applicable_rate(
            config["rate"],
            pan_available,
            specified_person
        )

        base_tds = amount * rate
        surcharge = base_tds * surcharge_rate
        cess = (base_tds + surcharge) * cess_rate
        total_tds = base_tds + surcharge + cess

        return {
            "nature": nature,
            "amount": amount,
            "rate": rate,
            "base_tds": round(base_tds, 2),
            "surcharge": round(surcharge, 2),
            "cess": round(cess, 2),
            "total_tds": round(total_tds, 2)
        }