# Backend Integration Guide

This document outlines the API integration setup for connecting the Frontend to a Java backend.

## Files Created

### 1. **src/config/api.ts** - API Configuration
Centralized configuration for API endpoints and base URL.
- `BASE_URL`: The Java backend URL (defaults to `http://localhost:8080/api`)
- `ENDPOINTS`: API endpoint paths
- `TIMEOUT`: Request timeout in milliseconds

### 2. **src/services/apiClient.ts** - HTTP Client
Generic REST API client that handles:
- HTTP requests (GET, POST, PUT, DELETE)
- Request timeout management
- Error handling
- JSON serialization/deserialization
- Headers management

### 3. **src/services/studentService.ts** - Student API Service
High-level service for student operations:
- `getAllStudents()` - GET /students
- `getStudentById(id)` - GET /students/{id}
- `createStudent(data)` - POST /students
- `updateStudent(id, data)` - PUT /students/{id}
- `deleteStudent(id)` - DELETE /students/{id}
- `searchStudents(query)` - GET /students/search?q={query}

## Configuration

### Environment Variables
Update the `.env` file with your Java backend URL:

```env
VITE_API_BASE_URL=http://localhost:8080/api
```

For development:
```env
VITE_API_BASE_URL=http://localhost:8080/api
```

For production:
```env
VITE_API_BASE_URL=https://api.yourdomain.com/api
```

## Java Backend Endpoints Required

Your Java backend should implement these REST endpoints:

### Students

#### Get All Students
- **Method**: GET
- **Endpoint**: `/api/students`
- **Response**: 
  ```json
  [
    {
      "id": "1",
      "name": "John Doe",
      "rollNo": "CS001",
      "email": "john@example.com",
      "phone": "+1234567890",
      "department": "Computer Science",
      "year": "3rd Year"
    }
  ]
  ```

#### Get Student by ID
- **Method**: GET
- **Endpoint**: `/api/students/{id}`
- **Response**: Single student object

#### Create Student
- **Method**: POST
- **Endpoint**: `/api/students`
- **Request Body**:
  ```json
  {
    "name": "John Doe",
    "rollNo": "CS001",
    "email": "john@example.com",
    "phone": "+1234567890",
    "department": "Computer Science",
    "year": "3rd Year"
  }
  ```
- **Response**: Created student object with ID

#### Update Student
- **Method**: PUT
- **Endpoint**: `/api/students/{id}`
- **Request Body**: Same as Create Student
- **Response**: Updated student object

#### Delete Student
- **Method**: DELETE
- **Endpoint**: `/api/students/{id}`
- **Response**: 200 OK (empty or success message)

#### Search Students
- **Method**: GET
- **Endpoint**: `/api/students/search?q={query}`
- **Query Parameters**: `q` - search query
- **Response**: Array of matching students

## Usage in Components

Replace the old `studentsAPI` (from `studentsData.ts`) with the new `studentService`:

### Before (Mock Data):
```typescript
import { studentsAPI } from '@/data/studentsData';

const data = studentsAPI.getAll();
```

### After (Real Backend):
```typescript
import { studentService } from '@/services/studentService';

const data = await studentService.getAllStudents();
```

## Error Handling

All service methods throw errors that should be caught and handled:

```typescript
try {
  const student = await studentService.createStudent(formData);
  toast.success('Student created successfully!');
} catch (error) {
  console.error('Failed to create student:', error);
  toast.error('Failed to create student');
}
```

## CORS Configuration

Make sure your Java backend has CORS enabled for the frontend URL:

```java
@Configuration
public class CorsConfig {
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("http://localhost:5173")); // Change to your frontend URL
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(Arrays.asList("*"));
        configuration.setAllowCredentials(true);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

## Migration from Mock Data

To switch from mock data to the real backend:

1. Update your components to use `studentService` instead of `studentsAPI`
2. Make methods async and use await
3. Wrap calls in try-catch for error handling
4. Use toast notifications for user feedback

Example update for StudentList.tsx:
```typescript
const loadStudents = async () => {
  try {
    const data = await studentService.getAllStudents();
    setStudents(data);
  } catch (error) {
    toast.error('Failed to load students');
  }
};
```

## Next Steps

1. Set up your Java backend with the required endpoints
2. Update `.env` with your backend URL
3. Update components to use `studentService`
4. Test the integration
