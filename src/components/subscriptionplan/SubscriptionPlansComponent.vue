<template>
    <div class="subscription-plans-page">
        <!-- Decorative Elements -->
        <div class="decorative-element decorative-element-1"></div>
        <div class="decorative-element decorative-element-2"></div>
        <div class="decorative-element decorative-element-3"></div>

        <!-- Loading indicator -->
        <div v-if="loading" class="loading-container">
            <div class="spinner"></div>
            <p>Cargando planes de suscripción...</p>
        </div>

        <!-- Error message -->
        <div v-else-if="error" class="error-container">
            <p>{{ error }}</p>
            <button class="btn-action btn-action-primary" @click="fetchSubscriptionPlans">
                Intentar nuevamente
            </button>
        </div>

        <div v-else>
            <!-- Period Toggle -->
            <div class="period-toggle">
                <button
                    @click="showMonthly"
                    :class="['toggle-option', { active: !yearly }]"
                >
                    Mensual
                </button>
                <button
                    @click="showYearly"
                    :class="['toggle-option', { active: yearly }]"
                >
                    Anual
                    <span class="saving-badge" v-if="yearly">Ahorre {{ yearlyDiscount }}%</span>
                    <span class="saving-badge saving-badge-promo" v-else>Ahorre {{ yearlyDiscount }}%</span>
                </button>
            </div>

            <!-- Plan Cards -->
            <div class="plans-container">
                <!-- Basic Plan -->
                <div class="subscription-plan-card" v-if="!yearly">
                    <div class="plan-header">
                        <span class="plan-badge">FREE</span>
                    </div>
                    <p class="plan-name">Plan básico inicial</p>
                    <div class="plan-pricing">
                        <span class="plan-price">$0</span>
                        <span class="plan-period">/ Por miembro</span>
                    </div>
                    <hr class="plan-divider">
                    <ul class="features-list">
                        <SubscriptionFeatureItem
                            v-for="(feature, index) in basicPlanFeatures"
                            :key="`basic-${index}`"
                            :feature-text="feature"
                        />
                    </ul>
                    <button class="btn-action btn-action-dark mt-auto" @click="startFreePlan">
                        Comenzar gratis
                    </button>
                    <p class="plan-footer-text">No se requiere tarjeta de débito/crédito</p>
                </div>

                <!-- Advanced Plan (Monthly) -->
                <div class="subscription-plan-card recommended" v-if="!yearly">
                    <div class="plan-header">
                        <span class="plan-badge">ADVANCED</span>
                        <span class="recommended-badge">RECOMENDADO</span>
                    </div>
                    <p class="plan-name">Plan profesional</p>
                    <div class="plan-pricing">
                        <span class="plan-price">${{ monthlyPrice }}</span>
                        <span class="plan-period">/ Por miembro</span>
                    </div>
                    <hr class="plan-divider">
                    <ul class="features-list">
                        <SubscriptionFeatureItem
                            v-for="(feature, index) in advancedPlanFeatures"
                            :key="`advanced-monthly-${index}`"
                            :feature-text="feature"
                        />
                    </ul>
                    <button class="btn-action btn-action-primary mt-auto" @click="startProPlan">
                        Comenzar prueba gratuita de 14 días
                    </button>
                    <p class="plan-footer-text">Se requiere tarjeta de débito/crédito</p>
                </div>
            </div>

            <!-- Plan Cards (Yearly) -->
            <div class="plans-container yearly-container" v-if="yearly">
                <!-- Advanced Plan (Yearly) -->
                <div class="subscription-plan-card recommended yearly-card">
                    <div class="plan-header">
                        <span class="plan-badge">ADVANCED</span>
                        <span class="recommended-badge">RECOMENDADO</span>
                    </div>
                    <p class="plan-name">Plan profesional</p>
                    <div class="plan-pricing">
                        <span class="plan-price">${{ yearlyPrice }}</span>
                        <span class="plan-period">/ Por miembro</span>
                    </div>
                    <hr class="plan-divider">
                    <ul class="features-list">
                        <SubscriptionFeatureItem
                            v-for="(feature, index) in advancedPlanFeatures"
                            :key="`advanced-yearly-${index}`"
                            :feature-text="feature"
                        />
                    </ul>
                    <button class="btn-action btn-action-primary mt-auto" @click="startYearlyPlan">
                        Comenzar prueba gratuita de 14 días
                    </button>
                    <p class="plan-footer-text">Se requiere tarjeta de débito/crédito</p>
                </div>
            </div>

            <!-- Payment Methods -->
            <div class="payment-methods-section">
                <p class="payment-methods-title">Métodos de pago</p>
                <div class="payment-icons">
                    <img src="/visa.png" alt="Visa" class="payment-icon">
                    <img src="/mastercard.png" alt="Mastercard" class="payment-icon">
                    <img src="/maestro.png" alt="Maestro" class="payment-icon">
                    <img src="/paypal.png" alt="Paypal" class="payment-icon">
                    <img src="/diners.png" alt="Diners Club" class="payment-icon">
                    <img src="/amex.png" alt="American Express" class="payment-icon">
                </div>
                <p class="payment-methods-footer">Aceptamos Visa, American Express, Mastercard y PayPal</p>
            </div>
        </div>
    </div>
</template>

<script>
import SubscriptionFeatureItem from '@/components/subscriptionplan/SubscriptionFeatureItem.vue';
import axios from 'axios';

export default {
    name: 'PatientSubscriptionPlansComponent',
    components: {
        SubscriptionFeatureItem
    },
    data() {
        return {
            basicPlanFeatures: [],
            advancedPlanFeatures: [],
            yearly: false,
            loading: true,
            error: null,
            monthlyPrice: 5.99,
            yearlyDiscount: 15 // Porcentaje de descuento
        }
    },
    computed: {
        yearlyPrice() {
            // Calcular precio anual con descuento 
            const discountedMonthly = this.monthlyPrice * (1 - this.yearlyDiscount / 100);
            return (discountedMonthly * 12).toFixed(2);
        }
    },
    created() {
        this.fetchSubscriptionPlans();
    },
    methods: {
        fetchSubscriptionPlans() {
            this.loading = true;
            this.error = null;
            
            axios.get(`${process.env.VUE_APP_API_URL}/subscription-plans`)
                .then(response => {
                    const plans = response.data;
                    
                    // Find basic plan (profesional_gratis)
                    const basicPlan = plans.find(plan => plan.subscription_type === 'profesional_gratis');
                    if (basicPlan && basicPlan.features) {
                        this.basicPlanFeatures = basicPlan.features.map(item => item.feature);
                    }
                    
                    // Find advanced plan (profesional_avanzado)
                    const advancedPlan = plans.find(plan => plan.subscription_type === 'profesional_avanzado');
                    if (advancedPlan && advancedPlan.features) {
                        this.advancedPlanFeatures = advancedPlan.features.map(item => item.feature);
                        if (advancedPlan.price_monthly) {
                            this.monthlyPrice = parseFloat(advancedPlan.price_monthly);
                        }
                        if (advancedPlan.discount_percent) {
                            this.yearlyDiscount = parseFloat(advancedPlan.discount_percent);
                        }
                    }
                })
                .catch(error => {
                    console.error('Error fetching subscription plans:', error);
                    this.error = 'Failed to load subscription plans';
                })
                .finally(() => {
                    this.loading = false;
                });
        },
        startFreePlan() {
            localStorage.setItem('selected_plan', 'gratis');
            localStorage.setItem('periodo', 'mensual');
            this.$router.push({
                name: 'Register',
            });
        },
        startProPlan() {
            localStorage.setItem('selected_plan', 'avanzado');
            localStorage.setItem('periodo', 'mensual');
            this.$router.push({
                name: 'Register',
            });
        },
        startYearlyPlan() {
            localStorage.setItem('selected_plan', 'avanzado');
            localStorage.setItem('periodo', 'anual');
            this.$router.push({
                name: 'Register',
            });
        },
        showMonthly() {
            this.yearly = false;
        },
        showYearly() {
            this.yearly = true;
        }
    }
}

</script>

<style scoped>
/* General Page Styles */
.subscription-plans-page {
    padding: 40px 20px;
    background-color: #F9FAFB; /* Light background consistent with other pages */
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    /* Added base font family */
    font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", "Liberation Sans", sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
    position: relative; /* Needed for absolute positioning of decorative elements */
    overflow: hidden; /* Prevent decorative elements from overflowing */
}

/* Loading Styles */
.loading-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    margin: 100px 0;
    z-index: 1;
}

.spinner {
    width: 50px;
    height: 50px;
    border: 5px solid rgba(13, 110, 253, 0.2);
    border-top: 5px solid #0d6efd;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-bottom: 20px;
}

@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}

/* Error Styles */
.error-container {
    background-color: #FEF2F2;
    border: 1px solid #F87171;
    border-radius: 8px;
    padding: 20px;
    margin: 40px 0;
    text-align: center;
    max-width: 500px;
    z-index: 1;
}

.error-container p {
    color: #B91C1C;
    margin-bottom: 15px;
    font-weight: 500;
}

/* Decorative Elements*/
.decorative-element {
    position: absolute;
    border-radius: 50%;
    opacity: 0.15; /* Adjusted opacity slightly */
    background: linear-gradient(45deg, #0d6efd, #86c7fe); /* Adjusted gradient for lighter feel */
    z-index: 0; /* Ensure they are behind content */
    pointer-events: none; /* Make them non-interactive */
}

.decorative-element-1 {
    width: 350px; /* Slightly larger */
    height: 350px;
    top: -175px;
    left: -175px;
}

.decorative-element-2 {
    width: 150px; /* Slightly larger */
    height: 150px;
    bottom: -50px; /* Adjusted position */
    right: 30%;
}

.decorative-element-3 {
    width: 250px; /* Slightly larger */
    height: 250px;
    bottom: -125px;
    right: -100px; /* Adjusted position */
}

/* Period Toggle */
.period-toggle {
    display: inline-flex;
    background-color: #F3F4F6; /* Light gray background for the toggle container */
    border-radius: 20px; /* Rounded corners for the toggle */
    padding: 4px;
    margin-bottom: 40px;
    box-shadow: inset 0 1px 3px rgba(0,0,0,0.05);
    position: relative; /* Ensure toggle is above decorative elements */
    z-index: 1;
}

.toggle-option {
    padding: 8px 20px;
    border: none;
    background-color: transparent;
    color: #6B7280; /* Default text color (gray) */
    font-weight: 500;
    border-radius: 16px; /* Rounded corners for buttons */
    cursor: pointer;
    transition: all 0.3s ease;
    font-size: 14px;
    display: flex;
    align-items: center;
    gap: 8px;
}

.toggle-option.active {
    background-color: #0d6efd; /* Changed to primary blue */
    color: white; /* White text for active */
    box-shadow: 0 2px 5px rgba(13, 110, 253, 0.2); /* Adjusted shadow color */
}

.saving-badge {
    background-color: #E7F1FF; /* Lighter blue background for saving badge */
    color: #0a58ca; /* Darker blue text */
    font-size: 12px;
    font-weight: 500;
    padding: 3px 8px;
    border-radius: 10px;
}

.saving-badge-promo {
    background-color: #e9ecef;
    color: #6c757d;
}

/* Plans Container */
.plans-container {
    display: flex;
    justify-content: center;
    gap: 30px;
    width: 100%;
    max-width: 1200px; /* Increased max-width for better side-by-side display */
    flex-wrap: wrap; /* Allow wrapping on smaller screens */
    position: relative; /* Ensure cards are above decorative elements */
    z-index: 1;
    flex-direction: row; /* Explicitly set to row to ensure side-by-side display */
}

/* Subscription Card Styling */
.subscription-plan-card {
    background-color: white;
    border-radius: 16px;
    padding: 30px;
    width: calc(50% - 30px); /* Set width to 50% minus gap for side-by-side display */
    max-width: 450px; /* Increased max width per card */
    min-width: 300px; /* Min width to ensure proper display */
    display: flex;
    flex-direction: column;
    box-shadow: 0 10px 30px rgba(0,0,0,0.07);
    border: 1px solid #E5E7EB; /* Subtle border */
    transition: transform 0.3s ease, box-shadow 0.3s ease;
    position: relative; /* Needed for recommended badge positioning */
    margin-bottom: 20px; /* Add margin bottom for spacing when stacked */
}

.subscription-plan-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 15px 35px rgba(0,0,0,0.1);
}

.subscription-plan-card.recommended {
     border-color: #0d6efd; /* Changed to primary blue */
     background-color: rgba(13, 110, 253, 0.03); /* Very subtle light blue background */
}

.plan-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
}

.plan-badge {
    display: inline-block;
    background-color: #F3F4F6;
    color: #374151;
    font-size: 12px;
    font-weight: 600;
    padding: 5px 12px;
    border-radius: 15px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
}

.recommended-badge {
    background-color: #0d6efd; /* Changed to primary blue */
    color: white;
    font-size: 11px;
    font-weight: 600;
    padding: 4px 10px;
    border-radius: 12px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
}

.plan-name {
    font-size: 18px;
    font-weight: 600; /* Made slightly bolder */
    color: #111827; /* Darker text */
    margin-bottom: 10px;
}

.plan-pricing {
    margin-bottom: 20px;
    display: flex;
    align-items: baseline; /* Align price and period */
}

.plan-price {
    font-size: 40px; /* Larger price */
    font-weight: 700; /* Bold price */
    color: #111827; /* Price color back to dark - can change if needed */
    margin-right: 5px;
    /* Removed specific font family override */
}

.plan-period {
    font-size: 14px;
    color: #6B7280;
    font-weight: 500;
}

.plan-divider {
    border: none;
    height: 1px;
    background-color: #E5E7EB;
    margin: 20px 0;
}

.features-list {
    padding-left: 0;
    list-style: none;
    margin-bottom: 25px;
    flex-grow: 1; /* Make list take available space */
}

/* Action Buttons */
.btn-action {
    width: 100%;
    padding: 12px 20px;
    border-radius: 8px;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    border: none;
    text-align: center;
}

.btn-action-dark {
    background-color: #374151; /* Dark gray */
    color: white;
    border: 1px solid #374151;
}

.btn-action-dark:hover {
    background-color: #1F2937; /* Darker gray */
    transform: translateY(-2px);
    box-shadow: 0 4px 10px rgba(55, 65, 81, 0.2);
}

/* Changed from gradient to solid blue */
.btn-action-primary {
    background-color: #0d6efd;
    color: white;
}

.btn-action-primary:hover {
    background-color: #0b5ed7; /* Darker blue on hover */
    transform: translateY(-2px);
    box-shadow: 0 4px 10px rgba(13, 110, 253, 0.3);
}

.plan-footer-text {
    font-size: 13px;
    color: #6B7280;
    text-align: center;
    margin-top: 15px;
    margin-bottom: 0;
}

/* Payment Methods Section */
.payment-methods-section {
    margin-top: 50px;
    text-align: center;
    width: 100%;
    max-width: 600px;
    position: relative; /* Ensure payment methods are above decorative elements */
    z-index: 1;
}

.payment-methods-title {
    font-size: 20px;
    font-weight: 600;
    color: #111827;
    margin-bottom: 20px;
    /* Removed specific font family override */
}

.payment-icons {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 15px; /* Space between icons */
    flex-wrap: wrap; /* Allow wrapping */
    margin-bottom: 15px;
}

.payment-icon {
    height: 25px;
    width: auto;
    opacity: 0.8;
}

.payment-methods-footer {
    font-size: 14px;
    color: #6B7280;
}

/* Invisible Plan Card */
.invisible {
    visibility: hidden;
    opacity: 0;
}

/* Yearly Plan Specific Styles */
.yearly-container {
    justify-content: center;
}

.yearly-card {
    max-width: 500px; /* Wider card for yearly plan */
    width: 80%; /* Use percentage to make it responsive */
}

/* Responsive Adjustments */
@media (max-width: 992px) {
    .plans-container {
        gap: 20px;
    }
    
    .subscription-plan-card {
        width: calc(50% - 20px); /* Adjust width for medium screens */
    }
}

@media (max-width: 768px) {
    .plans-container {
        flex-direction: column; /* Stack cards vertically */
        align-items: center;
        gap: 25px;
    }

    .subscription-plan-card {
        width: 100%; /* Full width on mobile */
        max-width: 90%; /* Allow slightly wider cards on mobile */
    }

    .yearly-card {
        width: 90%; /* Slightly wider on mobile */
        max-width: 90%;
    }

    .invisible {
        display: none; /* Hide placeholder on mobile */
    }

    .period-toggle {
        /* Make toggle slightly smaller if needed */
    }

    .toggle-option {
        padding: 7px 15px;
        font-size: 13px;
    }

    .saving-badge {
        font-size: 11px;
        padding: 2px 6px;
    }

    .plan-price {
        font-size: 36px;
    }
}

@media (max-width: 480px) {
    .subscription-plans-page {
        padding: 25px 10px;
    }

    .subscription-plan-card {
        padding: 20px;
        max-width: 100%;
    }

     .period-toggle {
        transform: scale(0.95); /* Slightly scale down toggle */
    }

     .plan-price {
        font-size: 32px;
    }

     .plan-period {
        font-size: 13px;
    }

    .btn-action {
        font-size: 14px;
        padding: 10px 15px;
    }

    .payment-icons {
        gap: 10px;
    }

    .payment-icon {
        height: 22px;
    }
}
</style>