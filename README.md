[18/05/2026 01:00] W✨ G:  #include <GL/glut.h>
#include <windows.h>
#include <math.h>

#ifndef M_PI
#define M_PI 3.14159265358979323846
#endif

// --- GLOBAL STATES ---
float logoX = 0.0f, logoY = 0.0f, logoAngle = 0.0f, logoScale = 1.0f;
float flagX = 0.0f, flagY = 0.0f, flagAngle = 0.0f, flagScale = 1.0f;
float waveTime = 0.0f;
float windScale = 0.5f;
float flagLift = 1.0f;

// States required by the systems
bool isNight = false;       // Toggles Day vs Night
bool isAnimated = true;     // Play/Pause master switch for all dynamic animations

// --- COLLABORATOR 4 STATES ---
float mouseX = 0.0f;
float mouseY = 0.0f;
float celestialHoverScale = 1.0f; // Scale modifier based on mouse detection
int sunColorShift = 0;            // Color mode changed via single clicks
int lastClickTime = 0;            // Tracker for double click calculation
float sunRotationAngle = 0.0f;    // Rotational transformation for solar rays
float sunDynamicPulse = 1.0f;     // Pulsing scalar transformation

// --- MOON TRANSLATION STATES ---
float moonTranslateX = 0.0f;      // Dynamic offset distance applied to Moon position
float moonTranslateY = 0.0f;      // Vertical glide translation path offset

// Forward declaration of dynamic path variables
float getCelestialX() { return 1100.0f + sin(waveTime * 0.2f) * 50.0f; }
float getCelestialY() { return 650.0f + cos(waveTime * 0.2f) * 20.0f; }

// ============================================================================
// COLLABORATOR 1 — Day/Night Background System
// ============================================================================

void renderBackgroundSystem() {
    if (isNight) {
        glClearColor(0.03f, 0.03f, 0.1f, 1.0f);
    } else {
        glClearColor(0.6f, 0.8f, 1.0f, 1.0f);
    }
    glClear(GL_COLOR_BUFFER_BIT);
}

// ============================================================================
// COLLABORATOR 2 — Sun and Moon Animation (with translation & matrix stacks)
// ============================================================================

void drawCelestialBody(float cx, float cy, float r, int segments, bool isSun) {
    glPushMatrix();

    // Base setup translation matrix targeting the core vector coordinates
    glTranslatef(cx, cy, 0.0f);

    if (!isSun && isNight) {
        // MOON TRANSLATION FEATURE: Modulate coordinate space translation in real-time
        glTranslatef(moonTranslateX, moonTranslateY, 0.0f);
    }

    // Calculate scaling metrics
    float activeScale = celestialHoverScale;
    if (isSun && !isNight) {
        activeScale *= sunDynamicPulse;
    }
    glScalef(activeScale, activeScale, 1.0f);

    // Draw Core Vector Geometry (Centered at 0,0 local matrix origins)
    glBegin(GL_POLYGON);
    for (int i = 0; i < segments; i++) {
        float theta = 2.0f * M_PI * float(i) / float(segments);
        glVertex2f(r * cosf(theta), r * sinf(theta));
    }
    glEnd();

    // Solar Rays Feature with Rotational Transformation
    if (isSun && !isNight) {
        glPushMatrix();
        glRotatef(sunRotationAngle, 0.0f, 0.0f, 1.0f);

        float rayPulse = isAnimated ? (sin(waveTime * 2.0f) * 0.1f + 0.9f) : 1.0f;
        float interactiveRedOffset = (mouseX / 1200.0f) * 0.3f;

        if (sunColorShift == 1)      glColor3f(1.0f, (0.2f + interactiveRedOffset) * rayPulse, 0.0f);
        else if (sunColorShift == 2) glColor3f(0.9f, 0.9f * rayPulse, 0.4f);
        else                         glColor3f(1.0f, (0.6f + interactiveRedOffset) * rayPulse, 0.0f);

        glLineWidth(2.5f);
        glBegin(GL_LINES);
        for (int i = 0; i < 16; i++) {
            float theta = 2.0f * M_PI * float(i) / 16.0f;
            glVertex2f(r * cosf(theta), r * sinf(theta));
            glVertex2f((r * 1.6f) * cosf(theta), (r * 1.6f) * sinf(theta));
        }
        glEnd();
        glPopMatrix();
    }
    glPopMatrix();
}

// ============================================================================
// COLLABORATOR 3 — Stars and Night Effects
// ============================================================================
[18/05/2026 01:00] W✨ G: 

void renderNightEffects() {
    if (!isNight) return;
    srand(777);

    glPointSize(1.0f);
    glBegin(GL_POINTS);
    for (int i = 0; i < 120; i++) {
        float x = (float)(rand() % 1200);
        float y = (float)(rand() % 750);
        float twinkle = isAnimated ? (sin(waveTime * 5.0f + i) * 0.4f + 0.6f) : 0.8f;
        glColor3f(twinkle * 0.7f, twinkle * 0.7f, twinkle * 0.9f);
        glVertex2f(x, y);
    }
    glEnd();

    glPointSize(3.0f);
    glBegin(GL_POINTS);
    for (int i = 0; i < 40; i++) {
        float x = (float)(rand() % 1200);
        float y = (float)(rand() % 750);
        float twinkle = isAnimated ? (cos(waveTime * 2.5f + (i * 15)) * 0.5f + 0.5f) : 1.0f;
        glColor3f(twinkle, twinkle, twinkle * 1.1f);
        glVertex2f(x, y);
    }
    glEnd();
}

// ============================================================================
// COLLABORATOR 4 — Mouse Interaction System
// ============================================================================

void handlePassiveMouseMotion(int x, int y) {
    mouseX = (float)x;
    mouseY = (float)(750 - y);

    float targetX = getCelestialX();
    float targetY = getCelestialY();

    // Adapt matrix displacement if tracking active translated Moon position
    if (isNight) {
        targetX += moonTranslateX;
        targetY += moonTranslateY;
    }

    float distance = sqrtf((mouseX - targetX) * (mouseX - targetX) + (mouseY - targetY) * (mouseY - targetY));

    if (distance < 90.0f) {
        celestialHoverScale = 1.35f;
    } else {
        celestialHoverScale = 1.0f;
    }
    glutPostRedisplay();
}

void handleMouseClicks(int button, int state, int x, int y) {
    if (button == GLUT_LEFT_BUTTON && state == GLUT_DOWN) {
        int currentTime = glutGet(GLUT_ELAPSED_TIME);
        int timeDiff = currentTime - lastClickTime;
        lastClickTime = currentTime;

        float targetX = getCelestialX();
        float targetY = getCelestialY();

        if (isNight) {
            targetX += moonTranslateX;
            targetY += moonTranslateY;
        }

        float dX = (float)x - targetX;
        float dY = (float)(750 - y) - targetY;
        float distance = sqrtf(dX * dX + dY * dY);

        if (distance < 100.0f) {
            if (timeDiff < 250) {
                isNight = !isNight;
            } else {
                sunColorShift = (sunColorShift + 1) % 3;
            }
        }
        glutPostRedisplay();
    }
}

// --- DRAWING HELPER FUNCTIONS ---
void drawRingArc(float cx, float cy, float radius, bool isFront) {
    glBegin(GL_LINE_STRIP);
    int start = isFront ? 270 : 90;
    int end = isFront ? 450 : 270;
    for (int i = start; i <= end; i++) {
        float rad = (float)i * M_PI / 180.0f;
        glVertex2f(cx + cos(rad) * radius, cy + sin(rad) * (radius * 0.4f));
    }
    glEnd();
}

void drawCircle(float cx, float cy, float r) {
    glBegin(GL_POLYGON);
    for (int i = 0; i < 360; i++) {
        float theta = 2.0f * M_PI * float(i) / 360.0f;
        glVertex2f(cx + r * cosf(theta), cy + r * sinf(theta));
    }
    glEnd();
}

// --- INTERACTION LOGIC (Including Play / Pause keys) ---
void handleKeypress(unsigned char key, int x, int y) {
    switch (key) {
        // PAUSE & RESUME KEY ASSIGNMENTS
        case 'p': isAnimated = false; break; // Pause all updates
        case 'c': isAnimated = true;  break; // Resume animations
[18/05/2026 01:00] W✨ G: 

        case 'w': logoY += 10.0f; break;
        case 's': logoY -= 10.0f; break;
        case 'a': logoX -= 10.0f; break;
        case 'd': logoX += 10.0f; break;
        case 'r': logoAngle += 5.0f; break;
        case 't': logoAngle -= 5.0f; break;
        case 'm': logoScale += 0.1f; break;
        case 'n': if (logoScale > 0.1) logoScale -= 0.1f; break;
        case 'y': flagY += 10.0f; break;
        case 'h': flagY -= 10.0f; break;
        case 'g': flagX -= 10.0f; break;
        case 'j': flagX += 10.0f; break;
        case 'i': flagAngle += 5.0f; break;
        case 'o': flagAngle -= 5.0f; break;
        case 'u': flagScale += 0.1f; break;
        case 'b': if (flagScale > 0.1) flagScale -= 0.1f; break;
        case '2': if (windScale < 3.0f) windScale += 0.1f; break;
        case '1': if (windScale > 0.1f) windScale -= 0.1f; break;
        case 'x': isNight = !isNight; break;
        case 27: exit(0); break;
    }
    glutPostRedisplay();
}

// --- DISPLAY SYSTEM ---
void display() {
    renderBackgroundSystem();
    renderNightEffects();

    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();

    // --- RENDER CELESTIAL ENVIRONMENT ---
    float pathX = getCelestialX();
    float pathY = getCelestialY();

    if (isNight) {
        float moonGlow = isAnimated ? (sin(waveTime * 0.5f) * 5.0f) : 0.0f;

        // Base structure drawing call handles moon local updates internally
        glColor3f(0.9f, 0.9f, 0.8f);
        drawCelestialBody(pathX, pathY, 45.0f + (moonGlow * 0.1f), 40, false);

        // Crescent Mask Overlay paired to follow identical Translation matrices
        glPushMatrix();
        glTranslatef(pathX + 20.0f, pathY + 15.0f, 0.0f);
        glTranslatef(moonTranslateX, moonTranslateY, 0.0f);
        glScalef(celestialHoverScale, celestialHoverScale, 1.0f);
        glColor3f(0.03f, 0.03f, 0.1f);
        drawCircle(0.0f, 0.0f, 45.0f);
        glPopMatrix();
    } else {
        if (sunColorShift == 1)      glColor3f(0.95f, 0.25f, 0.05f);
        else if (sunColorShift == 2) glColor3f(0.95f, 0.95f, 0.7f);
        else                         glColor3f(1.0f, 0.85f, 0.0f);

        drawCelestialBody(pathX, pathY, 50.0f, 50, true);
    }

    // --- FLAGPOLE AND OBJECT RENDERING ---
    glPushMatrix();
        glPushMatrix();
            glTranslatef(flagX, flagY, 0.0f);
            glTranslatef(150, 0, 0);
            glRotatef(flagAngle, 0, 0, 1);
            glScalef(flagScale, flagScale, 1);
            glTranslatef(-150, 0, 0);

            glColor3f(0.0, 0.0, 1.0);
            glBegin(GL_QUADS);
                glVertex2i(10, 0); glVertex2i(90, 0); glVertex2i(90, 30); glVertex2i(10, 30);
            glEnd();

            glColor3f(0.6, 0.1, 0.6);
            glBegin(GL_QUADS);
                glVertex2i(25, 30); glVertex2i(75, 30); glVertex2i(75, 55); glVertex2i(25, 55);
            glEnd();

            glColor3f(0.3, 0.3, 0.5);
            glBegin(GL_QUADS);
                glVertex2i(35, 55); glVertex2i(65, 55); glVertex2i(65, 75); glVertex2i(35, 75);
            glEnd();

            glColor3f(0.7, 0.7, 0.7);
            glBegin(GL_QUADS);
                glVertex2i(46, 75); glVertex2i(54, 75); glVertex2i(54, 840); glVertex2i(46, 840);
            glEnd();

            glColor3f(0.5, 0.5, 0.5);
            drawCircle(50, 845, 10);

            glColor3f(0.7, 0.7, 0.7);
            glLineWidth(3.0f);
            drawRingArc(50, 825.0f, 15.0f, true);
            drawRingArc(50, 640.0f, 15.0f, true);

            glColor3f(0.1, 0.1, 0.1);
            glLineWidth(2.5f);
            glBegin(GL_LINES);
                glVertex2f(65.0f, 825.0f); glVertex2f(65.0f, 220.0f);
                glVertex2f(65.0f, 825.0f); glVertex2f(62.0f, 825.0f);
                glVertex2f(65.0f, 640.0f); glVertex2f(62.0f, 640.0f);
                glVertex2f(60.0f, 220.0f); glVertex2f(65.0f, 220.0f);
            glEnd();
[18/05/2026 01:00] W✨ G: 

            glColor3f(0.2, 0.2, 0.2);
            glBegin(GL_LINE_LOOP);
                for(int i=0; i<360; i++) {
                    float rad = i*M_PI/180.0f;
                    glVertex2f(50.0f + cos(rad) * 10.0f, 220.0f + sin(rad) * 4.0f);
                }
            glEnd();

            for (float x = 62.0f; x < 480.0f; x += 1.0f) {
                float dist = x - 50.0f;
                float yW = (dist * (0.15f * windScale)) * sin(0.05f * x + waveTime);
                float xW = (dist * (0.03f * windScale)) * cos(0.05f * x + waveTime);

                if (x < 160.0f) glColor3f(1.0, 1.0, 1.0);
                else glColor3f(0.85, 0.0, 0.0);

                glBegin(GL_QUAD_STRIP);
                    glVertex2f(x + xW, 640.0f + yW);
                    glVertex2f(x + xW, 825.0f + yW);
                    float nx = x + 1.0f;
                    float nYW = ((nx - 50.0f) * (0.15f * windScale)) * sin(0.05f * nx + waveTime);
                    float nXW = ((nx - 50.0f) * (0.03f * windScale)) * cos(0.05f * nx + waveTime);
                    glVertex2f(nx + nXW, 640.0f + nYW);
                    glVertex2f(nx + nXW, 825.0f + nXW);
                glEnd();
            }

            glColor3f(1.0, 1.0, 1.0);
            glBegin(GL_TRIANGLES);
                float ywB = (110.0f * (0.15f * windScale)) * sin(8.0f + waveTime);
                float xwB = (110.0f * (0.03f * windScale)) * cos(8.0f + waveTime);
                float ywT = (170.0f * (0.15f * windScale)) * sin(11.0f + waveTime);
                float xwT = (170.0f * (0.03f * windScale)) * cos(11.0f + waveTime);

                for (float y = 640.0f; y < 825.0f; y += 37.0f) {
                    glVertex2f(160.0f + xwB, y + ywB);
                    glVertex2f(220.0f + xwT, y + 18.5f + ywT);
                    glVertex2f(160.0f + xwB, y + 37.0f + ywB);
                }
            glEnd();
        glPopMatrix();

        glPushMatrix();
            float cX = 750.0f, cY = 350.0f;
            glTranslatef(logoX + cX, logoY + cY, 0.0f);
            glRotatef(logoAngle, 0, 0, 1);
            glScalef(logoScale, logoScale, 1.0f);

            glColor3f(0.0, 0.51, 1.0);
            glBegin(GL_POLYGON);
                for (int i = 0; i <= 360; i++) {
                    float r = (float)i * M_PI / 180.0f;
                    glVertex2f(cos(r) * 110.0f, ((i > 180) ? -85.0f : 85.0f) + sin(r) * 110.0f);
                }
            glEnd();

            glColor3f(1.0f, 1.0f, 1.0f);
            glBegin(GL_QUADS);
                glVertex2f(-7.0f, -90.0f); glVertex2f(7.0f, -90.0f); glVertex2f(7.0f, 90.0f); glVertex2f(-7.0f, 90.0f);
                glVertex2f(-62.0f, -55.0f); glVertex2f(-48.0f, -55.0f); glVertex2f(47.0f, 40.0f); glVertex2f(33.0f, 40.0f);
                glVertex2f(-62.0f, 55.0f); glVertex2f(-48.0f, 55.0f); glVertex2f(47.0f, -40.0f); glVertex2f(33.0f, -40.0f);
                glVertex2f(-7.0f, 90.0f); glVertex2f(7.0f, 90.0f); glVertex2f(47.0f, 40.0f); glVertex2f(33.0f, 40.0f);
                glVertex2f(-7.0f, -90.0f); glVertex2f(7.0f, -90.0f); glVertex2f(47.0f, -40.0f); glVertex2f(33.0f, -40.0f);
            glEnd();
        glPopMatrix();
    glPopMatrix();

    glFlush();
}

void update() {
    // Check master flag layout setting state before modifying animation structures
    if (isAnimated) {
        waveTime += 0.005f * windScale;

        // 1. Angular Sun Spin Transformation
        sunRotationAngle += 0.25f;
        if (sunRotationAngle > 360.0f) sunRotationAngle -= 360.0f;

        // 2. Harmonic Sun Pulsing Scale Loop
        sunDynamicPulse = 1.0f + (sinf(waveTime * 3.5f) * 0.06f);

        // 3. MOON TRANSLATION CALCULATION: Creates continuous shifting drift vectors
        moonTranslateX = sinf(waveTime * 1.5f) * 60.0f;  // Shifting range bounds
        moonTranslateY = cosf(waveTime * 1.0f) * 15.0f;  // Vertical glide trajectory
    }

    glutPostRedisplay();
}
[18/05/2026 01:00] W✨ G: 

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB);
    glutInitWindowSize(1200, 750);
    glutInitWindowPosition(20, 20);
    glutIdleFunc(update);

    glutCreateWindow("Integrated Framework - Play/Pause and Moon Translation");

    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluOrtho2D(0.0, 1200.0, 0.0, 750.0);

    glutDisplayFunc(display);
    glutKeyboardFunc(handleKeypress);

    glutPassiveMotionFunc(handlePassiveMouseMotion);
    glutMouseFunc(handleMouseClicks);

    glutMainLoop();
    return 0;
}
