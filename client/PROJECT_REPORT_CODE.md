# Learning Management System - Important Code Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [Application Setup & Configuration](#application-setup--configuration)
3. [Authentication System](#authentication-system)
4. [State Management (Redux)](#state-management-redux)
5. [Course Management](#course-management)
6. [Lecture Management](#lecture-management)
7. [Payment Integration](#payment-integration)
8. [Role-Based Access Control](#role-based-access-control)
9. [Admin Dashboard](#admin-dashboard)
10. [Routing Configuration](#routing-configuration)

---

## Project Overview

This Learning Management System (LMS) is a full-stack web application built with React, Redux, and modern web technologies. It provides course creation, lecture management, payment integration, and role-based access control.

### Key Technologies
- **Frontend**: React 18.2.0, React Router DOM 6.21.1
- **State Management**: Redux Toolkit 2.0.1
- **HTTP Client**: Axios 1.6.3
- **UI**: Tailwind CSS 3.4.0, DaisyUI 4.4.24
- **Payment**: Razorpay Integration
- **Build Tool**: Vite 5.0.8

---

## Application Setup & Configuration

### 1. Main Entry Point (`src/main.jsx`)

```jsx
import './index.css'
import ReactDOM from 'react-dom/client'
import { Toaster } from 'react-hot-toast'
import { Provider } from 'react-redux'
import { BrowserRouter } from 'react-router-dom'

import App from './App.jsx'
import store from './redux/store.js'

ReactDOM.createRoot(document.getElementById('root')).render(
  <Provider store={store}>
    <BrowserRouter>
    <Toaster />
    <App />
    </BrowserRouter>
  </Provider>
)
```

**Key Features:**
- Redux store provider for global state management
- React Router for navigation
- Toast notifications for user feedback

### 2. Axios Configuration (`src/config/axiosInstance.js`)

```javascript
import axios from 'axios';

const BASE_URL = '/api/v1/'

const axiosInstance = axios.create({
    baseURL: BASE_URL,
    timeout: 10000,
    withCredentials: true
});

export default axiosInstance;
```

**Key Features:**
- Centralized API configuration
- Automatic credential handling for authentication
- Base URL configuration for API endpoints

---

## Authentication System

### 1. Authentication Slice (`src/redux/slices/authSlice.js`)

#### Initial State
```javascript
const initialState = {
    isLoggedIn: localStorage.getItem("isLoggedIn") || false,
    user: JSON.parse(localStorage.getItem("data"))?.data || {},
    role: localStorage.getItem("role") || "",
}
```

#### User Registration
```javascript
export const createAccount = createAsyncThunk("/auth/signup", async (data)=>{
    try {
        const responsePromise = axiosInstance.post("user/register",data);
        toast.promise(responsePromise,{
            loading:"Creating account...",
            success:(res)=>{
                return res.data?.message || "Promise Success , Accout Created";
            },
            error:(err)=>{
                return err?.response?.data?.message || "Promise is rejected , Unable to  Create Account";
            }
        })
        const response = await responsePromise;
        return response.data;
    } catch (error) {
        toast.error(error?.response?.data?.message || "Something went wrong");
        throw error;
    }
})
```

#### User Login
```javascript
export const login = createAsyncThunk('auth/login', async(data)=>{
    try {
        const responsePromise = axiosInstance.post("user/login",data);
        toast.promise(responsePromise,{
            loading:"Logging",
            success:(res)=>{
                return res.data?.message || " Promise Success , error in Handling the promise and LoggedIn  "
            },
            error:(err)=>{
                return err.response?.data?.message || "Promise is rejected in Login"
            }
        })
        return ( await responsePromise).data;
    } catch (error) {
        toast.error(error?.response?.data?.message || "unable to login");
        throw error;
    }
})
```

#### Login Reducer
```javascript
.addCase(login.fulfilled,(state,action)=>{
    localStorage.setItem("data",JSON.stringify(action?.payload));
    localStorage.setItem("isLoggedIn",true);
    localStorage.setItem("role",action?.payload?.data?.role);
    state.isLoggedIn=true;
    state.user=action?.payload?.data;
    state.role=action?.payload?.data?.role;
})
```

#### Password Reset Flow
```javascript
export const forgotPassword = createAsyncThunk('auth/forgotPassword', async (email) => {
    try {
        const responsePromise = axiosInstance.post('user/reset', { email });
        toast.promise(responsePromise, {
            loading: "Sending reset email...",
            success: (res) => {
                return res.data?.message || "Password reset email sent successfully";
            },
            error: (err) => {
                return err.response?.data?.message || "Failed to send reset email";
            }
        });
        return (await responsePromise).data;
    } catch (error) {
        toast.error(error?.response?.data?.message || "Unable to send password reset email");
        throw error;
    }
});

export const resetPassword = createAsyncThunk('auth/resetPassword', async (data) => {
    try {
        const responsePromise = axiosInstance.post(`user/reset/${data.resetToken}`, {
            password: data.password
        });
        toast.promise(responsePromise, {
            loading: "Resetting password...",
            success: (res) => {
                return res.data?.message || "Password reset successfully";
            },
            error: (err) => {
                return err.response?.data?.message || "Failed to reset password";
            }
        });
        return (await responsePromise).data;
    } catch (error) {
        toast.error(error?.response?.data?.message || "Unable to reset password");
        throw error;
    }
});
```

### 2. Sign In Component (`src/pages/SignIn.jsx`)

```javascript
const SignIn = () => {
    const navigate = useNavigate();
    const dispatch = useDispatch();

    const [signInDetails, setSignInDetails] = useState({
        email: '',
        password:''
    });

    async function onFormSubmit(e){
        e.preventDefault();
        if(!signInDetails.email || !signInDetails.password){
            toast.error("Please fill Email and Password");
            return;
        }

        if(!isEmail(signInDetails.email)){
            toast.error("Enter a Valid Email");
            return;
        }

        const response = await dispatch(login(signInDetails));
        if(response?.payload?.data){
            navigate("/");
        }
    }

    return (
        <HomeLayout>
            {/* Form JSX with validation and error handling */}
        </HomeLayout>
    )
}
```

**Key Features:**
- Email validation using regex
- Form validation before submission
- Toast notifications for user feedback
- Automatic navigation after successful login

---

## State Management (Redux)

### 1. Redux Store Configuration (`src/redux/store.js`)

```javascript
import { configureStore } from '@reduxjs/toolkit';

import authReducer from './slices/authSlice';
import courseReducer from './slices/courseSlice';
import razorPayReducer from './slices/razorPaySlice';
import lectureReducer from './slices/lectureSlice'

const store = configureStore({
    reducer:{
        auth: authReducer,
        course: courseReducer,
        razorpay: razorPayReducer,
        lecture : lectureReducer
    },
    middleware: (getDefaultMiddleware) => getDefaultMiddleware({
        serializableCheck: false
    }),
    devTools: true
});

export default store;
```

**Key Features:**
- Combined reducers for different features
- Disabled serializable check for handling File objects
- Redux DevTools enabled for debugging

---

## Course Management

### 1. Course Slice (`src/redux/slices/courseSlice.js`)

#### Initial State
```javascript
const initialState = {
    courseList:[],
}
```

#### Get All Courses
```javascript
export const getAllCourses = createAsyncThunk("/course/getAllCourses",async(data)=>{
    try {
        const responsePromise = axiosInstance.get('course', data);
        toast.promise(responsePromise, {
            loading: "Loading...",
            success: (res) => {
                return res.data?.message || "Promise Success , courseList fetched";
            },
            error: (err) => {
                return err.response?.data?.message || "Promise is rejected , Unable to fetch courseList";
            }
        })
        const response = await responsePromise;
        return response.data.data;
    } catch (err) {
        return console.log(err.response?.data?.message || "Something went wrong while fetching the courses");
    }
})
```

#### Create Course
```javascript
export const createNewCourse = createAsyncThunk("/course/create",async(data)=>{
    try {
        let formData = new FormData();
        formData.append("title",data?.title);
        formData.append("description",data?.description);
        formData.append("category",data?.category);
        formData.append("createdBy",data?.createdBy);
        formData.append("thumbnail",data?.thumbnail);

        const response = axiosInstance.post('course', formData);
        toast.promise(response,{
            loading:"Loading...",
            success:(res)=>{
                return res.data?.message || "Promise Success , course created"
            },
            error:(err)=>{
                return err.response?.data?.message || "Promise is rejected , Unable to create course"
            }
        })
        return (await response).data;
    } catch (err) {
        return console.log(err.response?.data?.message || "Something went wrong while creating the course")
    }
})
```

#### Update Course
```javascript
export const updateCourse = createAsyncThunk("/course/update", async (data) => {
    try {
        let formData = new FormData();
        formData.append("title", data?.title);
        formData.append("description", data?.description);
        formData.append("category", data?.category);
        formData.append("createdBy", data?.createdBy);
        if (data?.thumbnail) {
            formData.append("thumbnail", data?.thumbnail);
        }

        const responsePromise = axiosInstance.put(`course/${data.courseId}`, formData);
        toast.promise(responsePromise, {
            loading: "Updating course...",
            success: (res) => {
                return res.data?.message || "Course updated successfully";
            },
            error: (err) => {
                return err.response?.data?.message || "Failed to update course";
            }
        });
        return (await responsePromise).data;
    } catch (error) {
        console.log(error.response?.data?.message || "Something went wrong while updating the course");
        throw error;
    }
});
```

#### Delete Course
```javascript
export const deleteCourse = createAsyncThunk("/course/delete", async (courseId) => {
    try {
        const responsePromise = axiosInstance.delete(`course/${courseId}`);
        toast.promise(responsePromise, {
            loading: "Deleting course...",
            success: (res) => {
                return res.data?.message || "Course deleted successfully";
            },
            error: (err) => {
                return err.response?.data?.message || "Failed to delete course";
            }
        });
        return (await responsePromise).data;
    } catch (error) {
        console.log(error.response?.data?.message || "Something went wrong while deleting the course");
        throw error;
    }
});
```

#### Course Reducers
```javascript
const courseSlice = createSlice({
    name:"course",
    initialState,
    reducers: {} ,
    extraReducers: (builder)=>{
        builder
        .addCase(getAllCourses.fulfilled,(state,action)=>{
            if(action?.payload) {
                state.courseList = [...action.payload]
            }
        })
        .addCase(deleteCourse.fulfilled, (state, action) => {
            state.courseList = state.courseList.filter(course => course._id !== action.meta.arg);
        })
        .addCase(updateCourse.fulfilled, (state, action) => {
            const updatedCourse = action.payload.data;
            const courseIndex = state.courseList.findIndex(course => course._id === updatedCourse._id);
            if (courseIndex !== -1) {
                state.courseList[courseIndex] = updatedCourse;
            }
        })
        .addCase(createNewCourse.fulfilled, (state, action) => {
            if (action.payload?.data) {
                state.courseList.push(action.payload.data);
            }
        })
    }
})
```

### 2. Create Course Component (`src/pages/Course/CreateCourse.jsx`)

```javascript
const CreateCourse = () => {
    const dispatch = useDispatch();
    const navigate = useNavigate();

    const [ userInput, setUserInput ] = useState({
        title: "",
        description: "",
        createdBy: "",
        category: "",
        thumbnail: "",
        prevThumbnail: "",
        fileName: "",
    });

    const { getRootProps, getInputProps } = useDropzone({
        accept: ["image/jpeg", "image/png", "image/jpg", "image/svg", "image/gif"],
        onDrop: (acceptedFile)=>{
            const uploadImage = acceptedFile[0];
            const fileReader = new FileReader();
            fileReader.readAsDataURL(uploadImage);
            fileReader.onloadend = () => {
                setUserInput({
                    ...userInput,
                    thumbnail: uploadImage,
                    prevThumbnail: fileReader.result,
                    fileName: uploadImage.name,
                });
            }
        },
    });

    const handleSubmit = async (e) => {
        e.preventDefault();
        if(userInput.title === "" || userInput.description === "" || 
           userInput.createdBy === "" || userInput.thumbnail === ""){
            toast.error("Please fill all the fields");
            return;
        }

        if(userInput.description.length < 20){
            toast.error("Description should be atleast 20 characters long");
            return;
        }

        const response = await dispatch(createNewCourse(userInput));
        if(response.payload){
            navigate("/course");
        }
    }

    return (
        <HomeLayout>
            {/* Form with drag & drop image upload */}
        </HomeLayout>
    )
}
```

**Key Features:**
- Drag and drop image upload
- Form validation
- Image preview before upload
- File type validation

### 3. Course Description Component (`src/pages/Course/CourseDescription.jsx`)

```javascript
const CourseDescription = () => {
    const {state } = useLocation();
    const navigate = useNavigate();
    const { role, user } = useSelector((state)=>state.auth)

    return (
        <HomeLayout>
            <div className="min-h-[90vh] pt-16 pb-8 px-4 lg:px-8">
                {/* Course details display */}
                {role === "ADMIN" || user?.subscription?.status === "Active" ? (
                    <button onClick={() => navigate("/course/displayLecture", {state: {...state}})}>
                        Watch Lectures
                    </button>
                ) : (
                    <button onClick={() => navigate("/payment/subscribe")}>
                        Subscribe to Access
                    </button>
                )}
            </div>
        </HomeLayout>
    )
}
```

**Key Features:**
- Conditional rendering based on subscription status
- Role-based access control
- Course information display

---

## Lecture Management

### 1. Lecture Slice (`src/redux/slices/lectureSlice.js`)

#### Initial State
```javascript
const initialState = {
    lectures:[],
    currentLecture: localStorage.getItem('lastLecture') ? Number(localStorage.getItem('lastLecture')) : 0
}
```

#### Get Course Lectures
```javascript
export const getCourseLecture = createAsyncThunk( 'course/lecture/get' , async(courseId)=>{
    try {
        const response = axiosInstance.get(`/course/${courseId}`);
        toast.promise(response,{
            loading: 'Loading...',
            success: 'Lecture loaded',
            error: 'Error loading lecture'
        })  
        return (await response).data;
    } catch (error) {
        toast.error( error.response.data.message || 'Error loading lecture')
    }
});
```

#### Add Lecture
```javascript
export const addCourseLecture = createAsyncThunk('/course/lecture/add',async (data)=>{
    try {
        const formData = new FormData();
        formData.append('lecture', data.lecture);
        formData.append('title', data.title);
        formData.append('description', data.description);

        const response = axiosInstance.post(`/course/${data.courseId}`, formData);
        toast.promise(response,{
            loading: 'Loading...',
            success: 'Lecture added',
            error: 'Error adding lecture'
        })
        return (await response).data;
    } catch (error) {
        toast.error( error.response.data.message || 'Error adding lecture')
    }
});
```

#### Update Lecture
```javascript
export const updateCourseLecture = createAsyncThunk('/course/lecture/update', async (data) => {
    try {
        const formData = new FormData();
        formData.append('title', data.title);
        formData.append('description', data.description);
        if (data.lecture) {
            formData.append('lecture', data.lecture);
        }

        const response = axiosInstance.put(`/course/${data.courseId}/lectures/${data.lectureId}`, formData);
        toast.promise(response, {
            loading: 'Updating lecture...',
            success: 'Lecture updated successfully',
            error: 'Error updating lecture'
        });
        return (await response).data;
    } catch (error) {
        toast.error(error.response?.data?.message || 'Error updating lecture');
        throw error;
    }
});
```

#### Delete Lecture
```javascript
export const deletecourseLecture = createAsyncThunk('/course/lecture/delete', async (data)=>{
    try {
        const response = axiosInstance.delete(`/course/${data.courseId}/lectures/${data.lectureId}`);
        toast.promise(response,{
            loading: 'Loading...',
            success: 'Lecture deleted',
            error: 'Error deleting lecture'
        })
        return (await response).data;
    } catch (error) {
        toast.error( error.response.data.message || 'Error deleting lecture')
    }
});
```

#### Lecture Reducers
```javascript
const lectureSlice = createSlice({
    name: 'lecture',
    initialState,
    reducers:{
        setCurrentLecture: (state,action) =>{
            state.currentLecture = action.payload;
            localStorage.setItem('lastLecture', action.payload);
        }
    },
    extraReducers:(builder) => {
        builder
            .addCase(getCourseLecture.fulfilled, (state, action) => {
                state.lectures = action.payload?.data;
            })
            .addCase(addCourseLecture.fulfilled, (state, action) => {
                state.lectures.push(action.payload?.lecture);
            })
            .addCase(deletecourseLecture.fulfilled, (state, action) => {
                state.lectures = state.lectures.filter(lecture => lecture._id !== action.payload?.lectureId);
            })
            .addCase(updateCourseLecture.fulfilled, (state, action) => {
                state.lectures = action.payload?.data || state.lectures;
            })
    }
})
```

**Key Features:**
- Video file upload using FormData
- Lecture progress tracking
- CRUD operations for lectures

---

## Payment Integration

### 1. Razorpay Slice (`src/redux/slices/razorPaySlice.js`)

#### Initial State
```javascript
const initailState = {
    key:'',
    subscription_id:'',
    isPaymantSuccess:false,
    allPaymants:{},
    finalMonths:{},
    monthlySalesRecord:[]
}
```

#### Get Razorpay Key
```javascript
export const getRazorPayId = createAsyncThunk('/razorpay/getId',async ()=>{
    try {
        const response = await axiosInstance.get('/payment/razorpay-key');
        return response.data;
    } catch (error) {
        toast.error("failed to load data")
    }
});
```

#### Purchase Course Bundle
```javascript
export const purchaseCourseBundle = createAsyncThunk('/purchaseCourse',async()=>{
    try {
        const response = await axiosInstance.post('/payment/subscribe')
        return response.data;
    } catch (error) {
        toast.error(error?.response?.data?.message || 'payment failed')
    }
});
```

#### Verify Payment
```javascript
export const verifyUserPayment = createAsyncThunk('/payment/verify', async (data) => {
    try {
        const response = await axiosInstance.post('/payment/verify', {
            razorpay_payment_id: data.razorpay_payment_id,
            razorpay_subscription_id: data.razorpay_subscription_id,
            razorpay_signature: data.razorpay_signature,
        });
        return response.data;
    } catch (error) {
        toast.error("failed to verify payment")
    }
});
```

#### Cancel Subscription
```javascript
export const cancelCourseBundle = createAsyncThunk('/cancelCourse',async()=>{
    try {
        const response = axiosInstance.post('/payment/unsubscribe');
        toast.promise(response,{
            loading:"loading data",
            success:(res)=>{
                return res.data?.message || "Promise Success , Subscription Cancelled";
            },
            error:(err)=>{
                return err?.response?.data?.message || "Promise is rejected , Unable to  cancel subscription";
            }
        })
        return ( await response).data;
    } catch (error) {
        toast.error(error?.response?.data?.message || 'payment cancellation failed')
    }
})
```

### 2. Subscribe Component (`src/pages/Payment/Subscribe.jsx`)

```javascript
const Subscribe = () => {
    const dispatch = useDispatch()
    const navigate = useNavigate()

    const razorpayKey = useSelector((state) =>( state?.razorpay?.key))
    const subscription_id = useSelector((state )=> (state?.razorpay?.subscription_id));
    const isPaymantSuccess = useSelector((state) => (state?.razorpay?.isPaymantSuccess))

    async function handleSubscription(e){
        e.preventDefault()
        if(!razorpayKey){
            toast.error('Invalid Payment Gateway')
            return
        }

        try {
            const response = await dispatch(purchaseCourseBundle());
            
            if(!response.payload || !response.payload.subscription_id){
                toast.error("Failed to Create Subscription Id")
                return
            }

            const option = {
                key: razorpayKey,
                subscription_id: response.payload.subscription_id,
                name: "Git daemonX10",
                description: " No money will be deducted from your account",
                theme: {
                    color: '#3B82F6'
                },
                handler : async function (response ){
                    paymentDetails.razorpay_payment_id = response.razorpay_payment_id
                    paymentDetails.razorpay_subscription_id = response.razorpay_subscription_id
                    paymentDetails.razorpay_signature = response.razorpay_signature

                    toast.success('Payment Success');

                    const res = await dispatch(verifyUserPayment(paymentDetails));
                    res?.payload?.success ? navigate('/payment/subscribe/success') : navigate('/payment/subscribe/fail')
                }
            }
            
            const paymentOption = new window.Razorpay(option);
            paymentOption.open()
        } catch (error) {
            toast.error('An error occurred while processing your subscription')
        }
    }

    return (
        <HomeLayout>
            {/* Subscription form and payment gateway integration */}
        </HomeLayout>
    )
}
```

**Key Features:**
- Razorpay payment gateway integration
- Payment verification
- Subscription management
- Success/failure handling

---

## Role-Based Access Control

### 1. RequireAuth Component (`src/components/auth/RequireAuth.jsx`)

```javascript
import {useSelector} from 'react-redux'
import { Navigate, Outlet } from 'react-router-dom';

const RequireAuth = ({allowedRoles}) => {
    const { isLoggedIn , role } = useSelector((state) => state.auth);
    return isLoggedIn && allowedRoles.find((myrole)=>myrole=== role) ? (
        <Outlet />
    ) : isLoggedIn ? 
    (<Navigate to="/denied" />) : (<Navigate to="/login" />)
}

export default RequireAuth
```

**Key Features:**
- Role-based route protection
- Automatic redirection for unauthorized users
- Supports multiple roles per route

---

## Routing Configuration

### 1. App Component (`src/App.jsx`)

```javascript
import { Route,Routes } from "react-router-dom"
import RequireAuth from "./components/auth/RequireAuth"

const App = () => {
    return (
        <>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<AboutUs />} />
                <Route path="/signup" element={<SignUP/>} />
                <Route path="/login" element={<SignIn/>} />
                <Route path="/forgot-password" element={<ForgotPassword/>} />
                <Route path="/reset-password/:resetToken" element={<ResetPassword/>} />
                <Route path="/contact" element={<Contact />} />
                <Route path="/denied" element={< Denied />} />

                {/* User Routes */}
                <Route element={<RequireAuth allowedRoles={['ADMIN','USER']} />} >
                    <Route path="/user/profile" element={<Profile/>} />
                    <Route path="/user/editProfile" element={<EditProfile/>} />
                    <Route path="/request-admin" element={<RequestAdmin/>} />
                </Route>
                
                {/* Courses */}
                <Route path="/course" element={<CourseList />} />
                <Route path="/course/description" element={<CourseDescription />} />
                <Route path="/course/displayLecture" element={<DisplayLecture /> } />
                
                {/* Admin Routes */}
                <Route element={<RequireAuth allowedRoles ={["ADMIN"]} />}>
                    <Route path="/course/create" element={<CreateCourse/>} />
                    <Route path="/course/edit" element={<EditCourse/>} />
                    <Route path="/course/addLecture" element={<AddLecture />} />
                    <Route path="/admin/dashboard" element={<AdminDashboard />} />
                </Route>

                {/* Payment Routes */}
                <Route element={<RequireAuth allowedRoles ={["ADMIN","USER"]} />}>
                    <Route path="/payment/subscribe" element={<Subscribe />} />
                    <Route path="/payment/subscribe/success" element={<SubscribeSuccess />} />
                    <Route path="/payment/subscribe/fail" element={<SubscribeFail />} />
                </Route>
                
                <Route path="*" element={<NotFound/>} />
            </Routes>
        </>
    ) 
}

export default App
```

**Key Features:**
- Nested routing with protected routes
- Role-based access control
- Public and private routes separation
- 404 error handling

---

## Admin Dashboard

### 1. Admin Dashboard Component (`src/pages/Dashboard/AdminDashboard.jsx`)

```javascript
const AdminDashboard = () => {
    const dispatch = useDispatch();
    const navigate = useNavigate();
    const { courseList } = useSelector(state => state.course);
    const { user } = useSelector(state => state.auth);

    const handleDeleteCourse = async (courseId) => {
        if (window.confirm('Are you sure you want to delete this course?')) {
            try {
                const response = await dispatch(deleteCourse(courseId));
                if (response?.payload?.success) {
                    // Course removed from state by reducer
                }
            } catch (error) {
                toast.error('Failed to delete course');
            }
        }
    };

    const handleEditCourse = (course) => {
        navigate('/course/edit', { state: course });
    };

    const loadAdminRequests = async () => {
        setLoadingRequests(true);
        try {
            const response = await dispatch(getAdminRequests());
            if (response.payload?.data) {
                setAdminRequests(response.payload.data);
            }
        } catch (error) {
            toast.error('Failed to load admin requests');
        } finally {
            setLoadingRequests(false);
        }
    };

    const handleReviewRequest = async (requestId, status, reviewNotes = '') => {
        try {
            const response = await dispatch(reviewAdminRequest({
                requestId,
                status,
                reviewNotes
            }));
            if (response?.payload?.success) {
                await loadAdminRequests();
            }
        } catch (error) {
            toast.error('Failed to review request');
        }
    };

    return (
        <HomeLayout>
            <div className="min-h-[90vh] pt-16 pb-8 px-4 lg:px-8">
                {/* Statistics Cards */}
                <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
                    {/* Total Users, Subscribed Users, Total Courses, Total Revenue */}
                </div>

                {/* Course Management Table */}
                <div className="bg-white/10 backdrop-blur-sm border border-white/20 rounded-xl p-6">
                    <table className="w-full text-left">
                        <thead>
                            <tr>
                                <th>Course</th>
                                <th>Category</th>
                                <th>Instructor</th>
                                <th>Lectures</th>
                                <th>Actions</th>
                            </tr>
                        </thead>
                        <tbody>
                            {courseList?.map((course) => (
                                <tr key={course._id}>
                                    <td>{course.title}</td>
                                    <td>{course.category}</td>
                                    <td>{course.createdBy}</td>
                                    <td>{course.numberOfLectures}</td>
                                    <td>
                                        <button onClick={() => navigate('/course/description', { state: course })}>
                                            View
                                        </button>
                                        <button onClick={() => handleEditCourse(course)}>
                                            Edit
                                        </button>
                                        <button onClick={() => handleDeleteCourse(course._id)}>
                                            Delete
                                        </button>
                                    </td>
                                </tr>
                            ))}
                        </tbody>
                    </table>
                </div>

                {/* Admin Requests Management */}
                <div className="bg-white/10 backdrop-blur-sm border border-white/20 rounded-xl p-6 mt-8">
                    <h3>Admin Access Requests</h3>
                    {adminRequests.map((request) => (
                        <div key={request._id}>
                            <h4>{request.user.fullName}</h4>
                            <p>{request.reason}</p>
                            {request.status === 'pending' && (
                                <div>
                                    <button onClick={() => handleReviewRequest(request._id, 'approved')}>
                                        Approve
                                    </button>
                                    <button onClick={() => handleReviewRequest(request._id, 'rejected')}>
                                        Reject
                                    </button>
                                </div>
                            )}
                        </div>
                    ))}
                </div>
            </div>
        </HomeLayout>
    );
};
```

**Key Features:**
- Statistics dashboard
- Course management (CRUD operations)
- Admin request management
- User statistics
- Revenue tracking

---

## Layout Component

### 1. Layout Component (`src/layouts/Layout.jsx`)

```javascript
function HomeLayout ({ children }) {
    const navigate = useNavigate();
    const dispatch = useDispatch();

    const isLoggedIn = useSelector(state=>state?.auth?.isLoggedIn);
    const role = useSelector(state=>state?.auth?.role);

    async function onLogout(e){
        e.preventDefault();
        const res = await dispatch(logout());
        if(res?.payload?.success){
            navigate('/');
        }
    }

    return (
        <div className='min-h-[90vh]'>
            <div className='drawer'>
                <input type="checkbox" id='my-drawer' className='drawer-toggle' />
                <div className='drawer-content'>
                    {/* Menu button */}
                </div>
                <div className='drawer-side'>
                    <ul className='menu'>
                        {/* Navigation links */}
                        {isLoggedIn && role === 'ADMIN' && (
                            <>
                                <li><Link to='/course/create'>Create Course</Link></li>
                                <li><Link to='/admin/dashboard'>Admin Dashboard</Link></li>
                            </>
                        )}
                        {isLoggedIn ? (
                            <>
                                <li><Link to='/user/profile'>Profile</Link></li>
                                <li><button onClick={onLogout}>Logout</button></li>
                            </>
                        ) : (
                            <>
                                <li><Link to='/login'>Login</Link></li>
                                <li><Link to='/signup'>Sign Up</Link></li>
                            </>
                        )}
                    </ul>
                </div>
            </div>
            <div>
                {children}
                <Footer />
            </div>
        </div>
    )
}
```

**Key Features:**
- Responsive drawer navigation
- Role-based menu items
- Logout functionality
- Footer component

---

## Key Features Summary

### 1. Authentication & Authorization
- User registration and login
- Password reset functionality
- Role-based access control (Admin, User)
- Protected routes
- Session management with localStorage

### 2. Course Management
- Create, Read, Update, Delete courses
- Course thumbnail upload
- Course categorization
- Course description and details

### 3. Lecture Management
- Add, Update, Delete lectures
- Video file upload
- Lecture progress tracking
- Lecture description and metadata

### 4. Payment Integration
- Razorpay payment gateway
- Subscription management
- Payment verification
- Subscription cancellation

### 5. Admin Features
- Admin dashboard with statistics
- Course management
- Admin request approval system
- User management

### 6. User Features
- User profile management
- Course browsing
- Lecture viewing (with subscription)
- Admin access request

### 7. UI/UX Features
- Responsive design
- Toast notifications
- Loading states
- Error handling
- Modern UI with Tailwind CSS

---

## API Endpoints Used

### Authentication
- `POST /api/v1/user/register` - User registration
- `POST /api/v1/user/login` - User login
- `GET /api/v1/user/logout` - User logout
- `GET /api/v1/user/profile` - Get user profile
- `PUT /api/v1/user/update` - Update user profile
- `POST /api/v1/user/reset` - Forgot password
- `POST /api/v1/user/reset/:resetToken` - Reset password
- `POST /api/v1/user/request-admin` - Request admin access
- `GET /api/v1/user/admin-requests` - Get admin requests
- `PUT /api/v1/user/admin-requests/:requestId/review` - Review admin request

### Courses
- `GET /api/v1/course` - Get all courses
- `POST /api/v1/course` - Create course
- `PUT /api/v1/course/:courseId` - Update course
- `DELETE /api/v1/course/:courseId` - Delete course
- `GET /api/v1/course/:courseId` - Get course with lectures
- `POST /api/v1/course/:courseId` - Add lecture to course
- `PUT /api/v1/course/:courseId/lectures/:lectureId` - Update lecture
- `DELETE /api/v1/course/:courseId/lectures/:lectureId` - Delete lecture

### Payments
- `GET /api/v1/payment/razorpay-key` - Get Razorpay key
- `POST /api/v1/payment/subscribe` - Create subscription
- `POST /api/v1/payment/verify` - Verify payment
- `POST /api/v1/payment/unsubscribe` - Cancel subscription
- `POST /api/v1/payment?count=100` - Get payment records

---

## Environment Variables

```env
# API Configuration
VITE_API_BASE_URL=/api/v1/

# Razorpay Configuration (if needed on frontend)
VITE_RAZORPAY_KEY=your_razorpay_key
```

---

## Dependencies

### Core Dependencies
- `react`: ^18.2.0
- `react-dom`: ^18.2.0
- `react-router-dom`: ^6.21.1
- `redux`: @reduxjs/toolkit ^2.0.1
- `react-redux`: ^9.0.4
- `axios`: ^1.6.3

### UI Dependencies
- `tailwindcss`: ^3.4.0
- `daisyui`: ^4.4.24
- `framer-motion`: ^11.0.3
- `react-icons`: ^4.12.0
- `react-hot-toast`: ^2.4.1

### Additional Dependencies
- `react-dropzone`: ^14.2.3
- `react-player`: ^2.14.1
- `chart.js`: ^4.5.0
- `react-chartjs-2`: ^5.3.0

---

## Conclusion

This Learning Management System demonstrates a comprehensive full-stack application with:
- Modern React architecture with hooks and functional components
- Redux Toolkit for state management
- Role-based access control
- Payment gateway integration
- File upload handling
- Responsive UI design
- Error handling and user feedback
- RESTful API integration

The codebase follows best practices for React development, including:
- Component-based architecture
- Separation of concerns
- Reusable components
- Centralized state management
- Error handling
- User experience optimization

