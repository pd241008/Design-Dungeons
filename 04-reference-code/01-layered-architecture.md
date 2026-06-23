# Reference: The Perfect Layered Request

> **Concept Proven:** "Controllers -> validation only. Services -> ALL business logic."
> **Source Project:** Milan (`milanDSA-backend`)

This is the canonical example of how a route, controller, and service interact. By strictly enforcing this boundary, the controller is incredibly easy to test, and the service can be reused across different entry points (like a CLI or a queue worker).

## The Controller
Notice how the controller is barely 15 lines. It simply extracts the payload, passes it to the service, and wraps the response. 

**`src/controllers/auth_controller.ts`**
```typescript
import { Request, Response } from "express";
import { authService } from "../../services/auth/auth_service";

const authController = {
  /**
   * Triggers the OTP generation, hashing, and hybrid email delivery.
   */
  sendOtp: async (req: Request, res: Response) => {
    try {
      const { email } = req.body;
      const result = await authService.sendOtp(email);
      return res.json(result);
    } catch (e: any) {
      return res.status(400).json({ success: false, message: e.message });
    }
  },
  
  // ... verifyOtp, verifyPass
};

export default authController;
```

## The Service
The service is where the heavy lifting occurs. It handles the database (Supabase atomic upserts), crypto hashing, and rendering HTML for the email payload. If this logic was in the controller, it would be impossible to unit test the hashing logic without mocking an Express `req/res` object.

**`src/services/auth_service.ts`**
```typescript
import crypto from "crypto";
import path from "path";
import { supabase } from "../../config/supabase";
import { transporter } from "../../config/nodemailer";

export const authService = {
  async sendOtp(email: string) {
    if (!email) throw new Error("Email required");

    const otp = Math.floor(100000 + Math.random() * 900000).toString();
    const otpHash = crypto.createHash("sha256").update(otp).digest("hex");
    const expiresAt = new Date(Date.now() + 10 * 60 * 1000).toISOString();

    // 1. Atomic Upsert (Ensures hashing is saved to DB)
    const { error } = await supabase
      .from("otps")
      .upsert(
        { email, code: otpHash, expires_at: expiresAt },
        { onConflict: "email" },
      );

    if (error) throw new Error(error.message);

    // ... HTML rendering and transporter.sendMail() ...

    return { success: true };
  },
};
```
