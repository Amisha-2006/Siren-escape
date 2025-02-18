 using UnityEngine;

public class SirenController : MonoBehaviour
{
    public float jumpForce = 10f;
    private bool isJumping = false;
    private Rigidbody2D rb;

    // Start is called before the first frame update
    void Start()
    {
        rb = GetComponent<Rigidbody2D>(); // Get the rigidbody for physics
    }

    // Update is called once per frame
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space) && !isJumping)
        {
            Jump();
        }
    }

    void Jump()
    {
        isJumping = true;
        rb.velocity = Vector2.up * jumpForce; // Apply jump force
    }

    private void OnCollisionEnter2D(Collision2D collision)
    {
        if (collision.gameObject.CompareTag("Obstacle"))
        {
  GameManager.instance.GameOver(); // Trigger Game Over if collided with obstacle
        }
        else if (collision.gameObject.CompareTag("Ground"))
        {
            isJumping = false; // Reset jump when touching ground
        }
    }
}

public class ObstacleSpawner : MonoBehaviour
{
    public GameObject obstaclePrefab;
    public float spawnRate = 1f;
    public float minHeight = -2f, maxHeight = 3f;
    public float obstacleSpeed = 5f;

    private float timer = 0f;

    // Update is called once per frame
    void Update()
    {
        timer += Time.deltaTime;

        if (timer >= spawnRate)
        {
            SpawnObstacle();
            timer = 0f;
        }
    }

    void SpawnObstacle()
    {
        float spawnY = Random.Range(minHeight, maxHeight);
        Vector3 spawnPosition = new Vector3(10f, spawnY, 0); // Start at right side of screen
        GameObject obstacle = Instantiate(obstaclePrefab, spawnPosition, Quaternion.identity);
        obstacle.GetComponent<Obstacle>().SetSpeed(obstacleSpeed);
    }
}




public class Obstacle : MonoBehaviour
{
    private float speed;

    // Update is called once per frame
    void Update()
    {
        transform.Translate(Vector2.left * speed * Time.deltaTime); // Move leftwards
        if (transform.position.x < -10f) // If obstacle goes off screen
        {
            Destroy(gameObject); // Destroy the obstacle
        }
    }

    public void SetSpeed(float newSpeed)
    {
        speed = newSpeed;
    }
}





public class GameManager : MonoBehaviour
{
    public static GameManager instance;
    public Text scoreText;
    public Text gameOverText;
    private float score = 0f;
    private bool isGameOver = false;
    private float difficultyIncreaseTime = 30f;
    private float currentDifficultyTime = 0f;

    public float difficultySpeedMultiplier = 1.1f;
    private ObstacleSpawner obstacleSpawner;

    private void Awake()
    {
        instance = this;
        obstacleSpawner = FindObjectOfType<ObstacleSpawner>();
    }

    void Update()
    {
        if (!isGameOver)
        {
            score += Time.deltaTime; // Increase score as time progresses
            scoreText.text = "Score: " + Mathf.Floor(score).ToString…
            .ToString();

            // Increase difficulty after certain time intervals
            currentDifficultyTime += Time.deltaTime;
            if (currentDifficultyTime >= difficultyIncreaseTime)
            {
                obstacleSpawner.obstacleSpeed *= difficultySpeedMultiplier;
                currentDifficultyTime = 0f;
            }
        }
    }







    public void GameOver()
    {
        isGameOver = true;
        gameOverText.gameObject.SetActive(true);
        gameOverText.text = "Game Over! Final Score: " + Mathf.Floor(score).ToString();
    }

    public void RestartGame()
    {
        isGameOver = false;
        score = 0f;
        scoreText.text = "Score: 0";
        gameOverText.gameObject.SetActive(false);

        // Reset difficulty
        obstacleSpawner.obstacleSpeed = 5f;
    }
}




public class AudioManager : MonoBehaviour
{
    public AudioClip jumpSound;
    public AudioClip collisionSound;
    public AudioClip backgroundMusic;
    private AudioSource audioSource;

    void Start()
    {
        audioSource = GetComponent<AudioSource>();
        audioSource.loop = true;
        audioSource.clip = backgroundMusic;
        audioSource.Play(); // Play background music
    }

    public void PlayJumpSound()
    {
        audioSource.PlayOneShot(jumpSound);
    }

    public void PlayCollisionSound()
    {
        audioSource.PlayOneShot(collisionSound);
    }
}


