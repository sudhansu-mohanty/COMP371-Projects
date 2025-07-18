# Dynamic Lighting

The scene features a cube and 2 light sources.
The first is the camera,
The second is a rotating light source above the cube.

the camera can move with `W/A/S/D`.
While to look around you can use `Q/E` and the `arrows`.

## Sections that are responsible for the lghting

### Vertex Shader
The vertex shader is responsible for calculating the normal vectors and transforming vertex positions to world and clip spaces.

For the lighting we are only intrested in the normals

```glsl
    // Vertex and fragment shader sources (with lighting)
    const char* vertexShaderSource = R"(
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aNormal;
    layout (location = 2) in vec2 aTexCoord;

    out vec3 FragPos;
    out vec3 Normal;
    out vec2 TexCoord;

    uniform mat4 model;
    uniform mat4 view;
    uniform mat4 projection;

    void main()
    {
        FragPos = vec3(model * vec4(aPos, 1.0));
        Normal = mat3(transpose(inverse(model))) * aNormal;
        TexCoord = aTexCoord;
        gl_Position = projection * view * vec4(FragPos, 1.0);
    }
    )";
```

<br />

### Fragment Shader

The fragment shader calculates lighting per fragment using ambient, diffuse, and specular components from both light sources. It also incorporates shadow mapping to create realistic shadows from the rotating light source.

```glsl
    const char* fragmentShaderSource = R"(
    #version 330 core
    out vec4 FragColor;

    in vec3 FragPos;
    in vec3 Normal;
    in vec2 TexCoord;

    uniform vec3 lightPos[2];  // array for 2 light positions
    uniform vec3 viewPos;
    uniform sampler2D texture1;
    uniform sampler2D shadowMap;
    uniform mat4 lightSpaceMatrix;

    float ShadowCalculation(vec4 fragPosLightSpace, vec3 normal, vec3   lightDir);

    void main()
    {
        vec3 norm = normalize(Normal);
        vec3 viewDir = normalize(viewPos - FragPos);

        vec3 ambient = vec3(0.2);  // ambient is the same for both
        vec3 result = ambient;

        for (int i = 0; i < 2; i++) {
            vec3 lightDir = normalize(lightPos[i] - FragPos);

            // Diffuse
            float diff = max(dot(norm, lightDir), 0.0);
            vec3 diffuse = diff * vec3(1.0);

            // Specular
            float specularStrength = 0.5;
            vec3 reflectDir = reflect(-lightDir, norm);
            float spec = pow(max(dot(viewDir, reflectDir), 0.0), 32);
            vec3 specular = specularStrength * spec * vec3(1.0);

            // Combine for this light
            result += (diffuse + specular);
        }

        // Apply shadow from the first light only (optional — you can   extend this)
        vec4 fragPosLightSpace = lightSpaceMatrix * vec4(FragPos, 1.0);
        float shadow = ShadowCalculation(fragPosLightSpace, norm,   normalize(lightPos[0] - FragPos));

        result = (ambient + (1.0 - shadow) * (result - ambient)) *  texture(texture1, TexCoord).rgb;
        FragColor = vec4(result, 1.0);
    }

    float ShadowCalculation(vec4 fragPosLightSpace, vec3 normal, vec3   lightDir)
    {
        vec3 projCoords = fragPosLightSpace.xyz / fragPosLightSpace.w;
        projCoords = projCoords * 0.5 + 0.5;
        float closestDepth = texture(shadowMap, projCoords.xy).r;
        return step(projCoords.z, closestDepth);
    })";
```
<br />

### Dynamic Light Movement and Positon
- Light 0: Rotates around the cube using a sine and cosine function with time.

- Light 1: Fixed at the camera's position, moving as the camera moves.

```glsl
    // 1. Define two light positions
    glm::vec3 lightPos[2];

    // Light 0: rotating light
    float timeValue = glfwGetTime();
    float radius = 3.0f;
    glm::vec3 rotatingLightPos = glm::vec3(sin(timeValue) * radius, 2.0f, cos(timeValue) * radius);
    lightPos[0] = rotatingLightPos;

    // Light 1: static (e.g., camera position or fixed point)
    lightPos[1] = cameraPos;

    // 2. Set up shadow mapping using only light 0
    glm::mat4 lightProjection = glm::ortho(-10.0f, 10.0f, -10.0f, 10.0f, 1.0f, 7.5f);
    glm::mat4 lightView = glm::lookAt(lightPos[0], glm::vec3(0.0f), glm::vec3(0.0, 1.0, 0.0));
    glm::mat4 lightSpaceMatrix = lightProjection * lightView;
```

<br/>

### Light and Shadow Uniforms Sent to Shaders

```glsl
        glUniform3fv(glGetUniformLocation(shaderProgram, "lightPos"), 2, glm::value_ptr(lightPos[0]));
        glUniform3fv(glGetUniformLocation(shaderProgram, "viewPos"), 1, glm::value_ptr(cameraPos));
        glUniformMatrix4fv(glGetUniformLocation(shaderProgram, "lightSpaceMatrix"), 1, GL_FALSE, glm::value_ptr(lightSpaceMatrix));
```
<br />

### casting the shadow

```glsl
        glUseProgram(simpleDepthShaderProgram);
        glUniformMatrix4fv(glGetUniformLocation(simpleDepthShaderProgram, "lightSpaceMatrix"), 1, GL_FALSE, glm::value_ptr(lightSpaceMatrix));
        glBindFramebuffer(GL_FRAMEBUFFER, depthMapFBO);
        glViewport(0, 0, SHADOW_WIDTH, SHADOW_HEIGHT);
        glClear(GL_DEPTH_BUFFER_BIT);
        glm::mat4 model = glm::mat4(1.0f); // For the cube
        glUniformMatrix4fv(glGetUniformLocation(simpleDepthShaderProgram, "model"), 1, GL_FALSE, glm::value_ptr(model));
        glBindVertexArray(VAO); glDrawArrays(GL_TRIANGLES, 0, 36);
        glClearColor(0.1f, 0.1f, 0.1f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

        glUseProgram(shaderProgram);
```
<br />

## Failure

For some reason, my floor did not want to get rendered and when it did the cub dissapeared. Same behaviou when I enable mouse movement for the camera look around.

So I was not able to verify the shadows yet.

## Modification

To add additional light sources:

1. Increase the size of the `lightPos` array in both the shader and your application.

2. Modify the fragment shader to iterate over the new number of lights.

3. Adjust ambient, diffuse, and specular calculations accordingly.

4. Initialize and update the new light positions in your application code.

5. Pass updated light positions as uniforms to the shader.