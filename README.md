// Express route to handle plan id fetch 
router.get('/update-plan/:planId', async (req, res) => {
  try {
    const { planId } = req.params;

    const existingplan = await Plan.findById(planId);
console.log(existingplan,"existingplan");
if(!existingplan){
  return res.status(400).json({ message: 'Plan Not Available' });

}
  
    res.status(200).json(existingplan);
  } catch (error) {
    console.error('Error rejecting KYC:', error);
    res.status(500).json({ message: 'Error rejecting KYC', error: error.message });
  }
});
// Express route to handle plan updates
router.put('/update-plan/:planId', async (req, res) => {
  try {
    const { planId } = req.params;
    const { price, referalreward, earnings } = req.body;

    // Assuming you have a Plan model/schema defined in your application
    const updatedPlan = await Plan.findByIdAndUpdate(
      planId,
      { price, referalreward, earnings }, // Update the plan with the received data
      { new: true } // To return the updated plan after the update is applied
    );

    if (!updatedPlan) {
      return res.status(404).json({ message: 'Plan not found' });
    }

    res.status(200).json(updatedPlan);
  } catch (error) {
    console.error('Error updating plan:', error);
    res.status(500).json({ message: 'Error updating plan', error: error.message });
  }
});
---------------------------------------------------------------------------------------------
import {
  Card,
  CardBody,
  CardHeader,
  Col,
  Row,
  Form,
  FormGroup,
  Label,
  Input,
  Button,
} from 'reactstrap';

const validationSchema = yup.object().shape({
  earnings: yup
  .number()
    .typeError('Daily Earnings must be a number')
    .max(50, 'Daily Earnings should be below 50')
    .nullable(),

    referalreward: yup
    .number()
    .typeError('Referral Reward must be a number')
    .max(100, 'Referral Reward should be below 100')
    .nullable(),

  price: yup
  .number()
    .typeError('Price must be a number')
    .min(5, 'Plan Price should be greater than 5'),
});
const Plansview = () => {
  const { register, handleSubmit, setValue, formState: { errors } } = useForm({
    resolver: yupResolver(validationSchema),
        mode: "all"

  });
  const [plansData, setPlans] = useState([]);
  const [selectedPlan, setSelectedPlan] = useState(null); // State to hold selected plan details for editing

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await axios.get('http://localhost:3007/server/viewplan-admin');
        setPlans(response.data);
      } catch (error) {
        console.error('Error fetching plan details:', error);
      }
    };

    fetchData();
  }, []);

  const handlePlanClick = async (planId) => {
    try {
      const response = await axios.get(`http://localhost:3007/server/update-plan/${planId}`);
      setSelectedPlan(response.data); // Set selected plan details for editing
    } catch (error) {
      console.error('Error updating plan:', error);
    }
  };


  const handleInputChange = (e) => {
    const { name, value } = e.target;
    setSelectedPlan({ ...selectedPlan, [name]: value });
    setValue(name, value); // Update react-hook-form value
  };
  const onSubmit = async (data) => {
    try {
      console.log(data);
      await axios.put(`http://localhost:3007/server/update-plan/${selectedPlan._id}`, data);
    } catch (error) {
      console.error('Error updating plan:', error);
    }
  };
  return (
    <CCard className="mb-4">
      <CCardHeader>Step-Reward All Plans</CCardHeader>
      <CCardBody>
        <CRow>
          {plansData.map((plan, index) => (
            <CCol key={index} xs={12} sm={6} lg={4}>
              <CWidgetStatsB
                className="mb-4"
                color="success"
                inverse
                value={plan.price}
                title={plan.duration}
                progress={{ value: 89.9 }}
                text={
                  <>
                    <p>Lorem ipsum dolor sit amet enim.</p>
                    <h6>Daily Reward: $10</h6>
                  </>
                }
              />
              <button onClick={() => handlePlanClick(plan._id)}>Update Plan</button>
            </CCol>
          ))}
        </CRow>
        {selectedPlan && (
          <Form onSubmit={handleSubmit(onSubmit)}>
            <FormGroup>
              <Label for="price">Plan Price</Label>
              <Input
                type="text"
                id="price"
                name="price"
                {...register('price')}
                value={selectedPlan.price}
                onChange={handleInputChange}
              />
                {errors.price && <p>{errors.price.message}</p>}
            </FormGroup>
            <FormGroup>
              <Label for="earnings">Daily Earnings</Label>
              <Input
                type="text"
                id="earnings"
                name="earnings"
                {...register('earnings')}
                value={selectedPlan.earnings}
                onChange={handleInputChange}
              />
               {errors.earnings && <p>{errors.earnings.message}</p>}
            </FormGroup>
            <FormGroup>
              <Label for="referalreward">Referral Reward</Label>
              <Input
                type="text"
                id="referalreward"
                name="referalreward"
                {...register('referalreward')}
                value={selectedPlan.referalreward}
                onChange={handleInputChange}
              />
               {errors.referalreward && <p>{errors.referalreward.message}</p>}
            </FormGroup>
            <Button color="primary" type="submit">
              Submit
            </Button>
          </Form>
        )}
      </CCardBody>
    </CCard>
  );
};

export default Plansview;
